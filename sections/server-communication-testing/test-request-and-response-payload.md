# Test the request and response payloads

<br/><br/>

### One Paragraph Explainer

How many times the front-end application stops working because of a misaligned communication with the back-end?

The front-end application and the back-end one have a contract, and you always need to test contract compliance. Every communication between the two apps is defined by:

- its URL
- the HTTP verb used (GET, POST, etc.)
- the request payload and headers: the data that the front-end application sends to the back-end one
- the response payload, headers, and status: the data that the back-end application sends back to the front-end one

You need to test all of them and, more in general, you need to wait for every relevant AJAX request, why?

- a relevant XHR request is part of the application flow that you're testing
- if an XHR request is not part of the flow that you're testing, it could be relevant to reach the desired UI state
- waiting for XHR requests make your test more robust, see the [Await, don't sleep](/sections/generic-best-practices/await-dont-sleep.md) chapter and its [XHR request waitings](/sections/generic-best-practices/await-dont-sleep.md#xhr-request-waitings) section

Full XHR request waiting and inspection is not so common in the existing testing tools, Cypress provides the most complete inspection support at the moment.

<br/><br/>

*Please note: all the following examples are for Cypress, it has the best XHR testing support at the moment.*

## Asserting about an XHR request, a complete example

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

In the next sections, we are going to split the different characteristics of an XHR request.





<details><summary>Asserting about the XHR request URL</summary>

With Cypress, the URL used for the request is defined with the `cy.intercept` call. You could need to inspect the query string of the URL.

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

Please note that the `then => expect` syntax of Cypress is helpful when you need to assert about multiple subjects (ex. both the URL and the status). If you need to assert about a single subject you could use more expressive `should` syntax

```javascript
cy.wait("@authentication-xhr")
  .its("url")
  .should("contain", "username=admin")
  .and("contain", "password=asupersecretpassword");
```
</details>





<details><summary>The XHR request method</summary>

With Cypress, the method used for the request is defined calling the `cy.intercept` function. You specify it to define what kind of request you want to intercept.

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





<details><summary>Asserting about the XHR request payload and headers</summary>

Asserting about the request payload and headers allows you to have immediate and detailed feedback about the reason for a bad XHR request. They must be checked on every single XHR request to be sure that everything represents correctly the UI actions the test makes.

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




<details><summary>Asserting about the XHR response payload, headers, and status</summary>

The response must adhere 100% to what the front-end application expects, otherwise, an unexpected state could be shown to the user. Response assertions are useful for full E2E tests, while they're useless in UI integration tests (TODO: link the integration test page).

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
