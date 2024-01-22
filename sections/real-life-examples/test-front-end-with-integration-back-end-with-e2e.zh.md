# 真实案例：*用集成测试测试前端，用 E2E 测试测试后端 - 参考 [组件测试 vs（UI）集成测试 vs E2E 测试](./sections/testing-strategy/component-vs-integration-vs-e2e-testing.zh.md)

<br/><br/>

## 一段简要说明

UI tests with a stubbed server are [highly reliable and faster](../testing-strategy/component-vs-integration-vs-e2e-testing.zh==.md#ui-integration-tests)<!--TODO: check that the deeplinkl works--> in comparison to full E2E tests.

Full E2E tests still provide the highest possible confidence, but at a high cost: being brittle, potentially unreliable, and slow.

We can still achieve high confidence for the front-end by using lower-cost UI integration tests and saving higher cost E2E tests for the back-end.

使用带有存根服务器的 UI 测试相对于完整的 E2E 测试来说，不仅更加[可靠，而且速度更快](../testing-strategy/component-vs-integration-vs-e2e-testing.zh.md#UI-集成测试)。<!--TODO: 检查深链接是否有效-->

尽管完整的 E2E 测试仍然提供了最高的信心水平，但代价很高：易碎、潜在不可靠且速度较慢。

通过使用成本较低的 UI 集成测试，我们仍然可以在前端获得高信心水平，并将成本较高的 E2E 测试留给后端。

<br/><br/>

## 示例测试架构图

这是来自实际的[建筑控制云应用程序](https://new.siemens.com/global/en/products/buildings/digitalization/building-operator.html)的高层次架构视图。

* Angular 前端
* Node-Express API（后端）
* 服务（Go lambdas）（后端）
* 硬件（后端）

> 根据需求，可以通过纯 API 测试来补充 UI-E2E 测试。一些常用的 API 测试工具包括 [Postman](https://www.getpostman.com/)，[VS Code 的 Rest Client](https://marketplace.visualstudio.com/items?itemName=humao.rest-client)，以及 Cypress。

![ ](./../../assets/images/test-architecture-example.png)

<br/>

*请注意：以下所有示例均使用 Cypress，目前它在 XHR 测试支持方面是最优秀的。[在现有的测试工具中，完整的 XHR 请求等待和检查并不常见](../generic-best-practices/await-dont-sleep.zh.md#XHR-请求等待)，Cypress 目前提供了最全面的检查支持。*

<br/>

## 代码示例：登录测试

以下示例包含两个测试，用于覆盖登录功能。第一个测试使用 UI 集成测试覆盖前端应用程序，第二个测试使用 E2E 测试覆盖后端。

```javascript
/** function to fill username, password and Login*/
const fillFormAndClick = ({ username, password }) => { .. };

// This is an UI integration test with server stubbing.
// Remember to write a few E2E tests and a lot of integration ones
// @see https://github.com/NoriSte/ui-testing-best-practices/blob/master/sections/testing-strategy/component-vs-integration-vs-e2e-testing.md#ui-integration-tests
it("Login: front-end integration tests", () => {

  // A route that intercepts / sniffs every POST request that goes to the authentication URL.
  // Stubs the response with authentication-success.json fixture. This is called server stubbing
  cy.intercept({
    method: "POST",
    fixture: "authentication/authentication-success.json", // Stubs the response
    url: `**${AUTHENTICATE_API_URL}`
  }).as("auth-xhr");

  fillFormAndClick(USERNAME_PLACEHOLDER, PASSWORD_PLACEHOLDER);

  // wait for the POST XHR
  cy.wait("@auth-xhr").then(interception => {
    // assert the payload body that the front end is sending to the back-end
    expect(interception.request.body).to.have.property("username", username);
    expect(interception.request.body).to.have.property("password", password);
    // assert the request headers in the payload
    expect(interception.request.headers).to.have.property('Content-Type', 'application/json;charset=utf-8');
  });

  // finally, the user must see the feedback
  cy.getByText(SUCCESS_FEEDBACK).should("be.visible");
});

// this is a copy of the integration test but without server stubbing.
it("Login: back-end E2E tests", () => {

  // A route that intercepts / sniffs every POST request that goes to the authentication URL.
  // Distinction: this is NOT stubbed!
  cy.intercept({
    method: "POST",
    url: `**${AUTHENTICATE_API_URL}`
  }).as("auth-xhr");

  fillFormAndClick(USERNAME_PLACEHOLDER, PASSWORD_PLACEHOLDER);

  cy.wait("@auth-xhr").then(interception => {
    // since the integration tests already tested the front-end app, we use E2E tests to check the
    // back-end app. It needs to ensure that the back-end app works and gets the correct response data

    // response body assertions and status should be in the E2E tests since they rely on the server
    expect(interception.status).to.equal(200);
    expect(interception.response.body).to.have.property("token");
  });

  // finally, the user must see the feedback
  cy.getByText(SUCCESS_FEEDBACK).should("be.visible");
});
```

<br/><br/>

## 代码示例 2：将 UI 集成测试切换为 UI-E2E 测试

有时候，您可能希望将 UI 集成测试切换为 E2E 测试。

在较低级别的测试层面，例如仅将测试隔离到 UI 代码时，您可能更愿意使用经济高效的 UI 集成测试。

而在较高级别的测试层面，例如与中间层服务集成时，您可能需要更高的信心水平，将测试目标定位到后端。

通过使用条件存根，您可以在 UI 集成测试和 E2E 测试之间切换关注。

```javascript
// stub-services.js : a file that only includes a function to stub the back-end services
export default function() {
  // all routes to the specified endpoint will respond with pre-packaged Json data
  cy.intercept('/api/../me', {fixture:'services/me.json'});
  cy.intercept('/api/../permissions', {fixture:'services/permissions.json'});
  // Lots of other fixtures ...
}

// spec file:
import stubServices from '../../support/stub-services';

/** isLocalhost is a function that checks the configuration environment*/
const isLocalHost = () => Cypress.env('ENVIRONMENT') === "localhost";

// ... in your tests, or in before / beforeEach blocks,
// stub the services if you are testing front-end (UI integration tests)
// do not stub if you are testing the back-end (UI-E2E tests)
if (isLocalHost()) {
  stubServices();
}

```

## 参考资料

[熟练掌握 UI 测试 - 会议视频](https://www.youtube.com/watch?v=RwWz4hllDtg)
<!-- TODO: 最后，决定是否将所有资源移动到一个共同的章节中。-->
