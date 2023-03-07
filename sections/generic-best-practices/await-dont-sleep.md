# Await, don't sleep

<br/><br/>

### One Paragraph Explainer

When testing your UI, you define a sort of key points the app must pass through. Reaching these key
points is an asynchronous process because, almost 100% of the times, your UI does not update
synchronously.

Those key points are called **deterministic events**, as known as something that you know that must happen.

It depends on the events you define and how your UI reaches them but, usually, there are some
"long" waitings, like XHR requests, and some faster ones, like re-render updates.

The solution to the asynchronous updates seems handy: **sleeping/pausing the test** for a bunch of
milliseconds, tenths of a second, or even seconds. It can make your test working because it gives
the app the time to update itself and moving to the next deterministic event to be tested.

Consider that, except for specific and known waitings (like when you use `setInterval` or
`setTimeout`), **it's totally unpredictable** how much the sleeping time should be because it could depend on:

- the network state (for XHR requests)
- the total amount of available machine resources (CPU, RAM, etc.)
  - a CI pipeline can limit them for example
  - other apps can consume them on your local machine too
- the concurrence of other resource-consuming updates (canvas, etc.)
- a bunch of unpredictable game players like Service Workers, cache management etc. that can make
  the UI update process faster or slower

Every fixed delay leads your test to be more brittle and **increasing its duration**. You are going to
find a balance between false negatives—when the test fails because of a too low sleeping
time—and exaggerate test duration.

What about waiting just the right amount of time? The amount of time that makes the test as fast as
possible!

Waitings fall in four main categories

- **[page load waitings](#page-load-waitings)**: the first waiting to manage while testing your app, waiting for an event that
  allows you to understand that the page is interactive
- **[content waitings](#content-waitings)**: waiting for DOM element that matches a selector
- **[XHR request waitings](#xhr-request-waitings)**: waiting for an XHR request start or the corresponding response received

All the following examples are based on Cypress.

<br/><br/>

## Page load waitings

```javascript
// Cypress code
cy.visit('http://localhost:3000')
```

## Content waitings

Take a look at the following examples to see how waiting for a DOM element could be implemented in
the available tools.

### Code Examples

- waiting for an element:

```javascript
// Cypress code

// it waits up to 4 seconds by default
cy.get('#form-feedback')
// the timeout can be customized
cy.get('#form-feedback', { timeout: 5000 })
```

- waiting for an element with specific content

```javascript
// Cypress code

cy.get('#form-feedback').contains('Success')
```

## XHR request waitings

### Code Examples

- waiting for an XHR request/response

```javascript
// Cypress code

cy.intercept('http://dummy.restapiexample.com/api/v1/employees').as('employees')
cy.wait('@employees')
  .its('response.body')
  .then((body) => {
    /* ... */
  })
```

<br /><br />

_Crossposted by [NoriSte](https://github.com/NoriSte) on [dev.to](https://dev.to/noriste/await-do-not-make-your-e2e-tests-sleep-4g1o) and [Medium](https://medium.com/@NoriSte/react-hooks-memorandum-bf1c2758a672)._
