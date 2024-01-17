# 检验请求和响应负载

<br/><br/>

## 一段简要说明

前端应用因与后端通信不协调而导致停止工作的频率有多高？

前端应用和后端应用之间存在一份合同，您始终需要测试合同是否得到遵守。每一次前后端应用之间的通信都由以下几个方面定义：

- 请求的 URL
- 所使用的 HTTP 动词（GET、POST 等）
- 请求的有效负载和标头：前端应用发送给后端应用的数据
- 响应的有效负载、标头和状态：后端应用发送回前端应用的数据

您需要对所有这些方面进行测试，更广义地说，您需要等待每个相关的 AJAX 请求，为什么呢？

- 相关的 XHR 请求是您正在测试的应用程序流程的一部分
- 即使 XHR 请求不是您正在测试的流程的一部分，它对达到期望的 UI 状态也可能是相关的
- 等待 XHR 请求可以使您的测试更为健壮，参见[等待，不要休眠](/sections/generic-best-practices/await-dont-sleep.zh.md)章节及其[XHR 请求等待](/sections/generic-best-practices/await-dont-sleep.zh.md#XHR-请求等待)部分

在现有的测试工具中，完全等待和检查 XHR 请求并不那么常见，目前 Cypress 提供了最全面的检查支持。

<br/><br/>

*请注意：以下所有示例均基于 Cypress，它目前提供了最佳的 XHR 测试支持。*

## 对 XHR 请求进行断言的完整示例

```javascript
// ask Cypress to intercept every XHR request made to a URL ending with `/authentication`
cy.intercept("POST", "**/authentication").as("authentication-xhr");

// ... your test actions...

cy.wait("@authentication-xhr").then(interception => {
  // request headers assertion
  expect(interception.request.headers).to.have.property("Content-Type", "application/json");
  // request payload assertions
  expect(interception.request.body).to.have.property("username", "admin");
  expect(interception.request.body).to.have.property("password", "asupersecretpassword");
  // status assertion
  expect(interception.response.statusCode).to.equal(200);
  // response headers assertions
  expect(interception.response.body).to.have.property("access-control-allow-origin", "*");
  // response payload assertions
  expect(interception.response.body).to.have.property("token");
});
```

在下面的章节中，我们将详细讨论 XHR 请求的不同特征。

<details><summary>验证 XHR 请求的 URL</summary>

在 Cypress 中，用于请求的 URL 是通过`cy.intercept`调用定义的。您可能需要检查 URL 的查询字符串。

```javascript
// ask Cypress to intercept every XHR request made to a URL ending with `/authentication`
cy.intercept("**/authentication**").as("authentication-xhr");

// ... your test actions...

cy.wait("@authentication-xhr").then(interception => {
  // query string assertion
  expect(interception.request.url).to.contain("username=admin");
  expect(interception.request.url).to.contain("password=asupersecretpassword");
});
```

请注意，当您需要对多个主题进行断言时，Cypress 的`then => expect`语法非常有帮助（例如，URL 和状态）。如果您只需要对单个主题进行断言，可以使用更具表现力的`should`语法。

```javascript
cy.wait("@authentication-xhr")
  .its("url")
  .should("contain", "username=admin")
  .and("contain", "password=asupersecretpassword");
```

</details>

<details><summary>XHR 请求的方法</summary>

在 Cypress 中，请求使用`cy.intercept`函数定义。您可以通过指定它来定义要拦截的请求类型。

```javascript
// the most compact `cy.intercept` call, the GET method is implied
cy.intercept("**/authentication").as("authentication-xhr");

// method can be explicitly defined
cy.intercept("POST", "**/authentication").as("authentication-xhr");

// the extended `cy.intercept` call is available too
cy.intercept({
  method: "POST",
  url: "**/authentication"
}).as("authentication-xhr");
```

</details>

<details><summary>验证 XHR 请求的 payload 和 headers</summary>

对 XHR 请求的 payload 和 headers 进行断言允许您立即获得有关糟糕的 XHR 请求原因的详细反馈。必须在每个 XHR 请求上进行检查，以确保一切都正确地表示了测试执行的 UI 操作。

```javascript
// ask Cypress to intercept every XHR request made to a URL ending with `/authentication`
cy.intercept("POST", "**/authentication").as("authentication-xhr");

// ... your test actions...

cy.wait("@authentication-xhr").then(interception => {
  // request headers assertion
  expect(interception.request.headers).to.have.property("Content-Type", "application/json");
  // request payload assertions
  expect(interception.request.body).to.have.property("username", "admin");
  expect(interception.request.body).to.have.property("password", "asupersecretpassword");
});
```

</details>

<details><summary>验证 XHR 请求响应的 payload, headers 和 status</summary>

响应必须百分之百符合前端应用的预期，否则可能向用户展示意料之外的状态。响应断言在完整的端到端测试中很有用，但在 UI 集成测试中则无关紧要（TODO：链接到集成测试页面）。

```javascript
// ask Cypress to intercept every XHR request made to a URL ending with `/authentication`
cy.intercept("POST", "**/authentication").as("authentication-xhr");

// ... your test actions...

cy.wait("@authentication-xhr").then(intercept => {
  // status assertions
  expect(intercept.response.statusCode).to.equal(200);
  // response headers assertions
  expect(intercept.response.body).to.have.property("access-control-allow-origin", "*");
  // response payload assertions
  expect(intercept.response.body).to.have.property("token");
});
```

</details>
