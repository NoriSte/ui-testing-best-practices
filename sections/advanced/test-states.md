# Test States

<br/><br/>

### One Paragraph Explainer

Tests should be repeatable, modular and should handle their own state setup. UI Tests should not be repeated in order to achieve state for another test.

We want to have stateless that can scale:

* Tests that independently handle their state.
* Have no side effects outside themselves, or manageable side effects which the test handles itself
* Tests that can be executed by *n* number of entities simultaneously.

<br/><br/>

### Code Example – explanation

**Repeatability**: tests must setup state, perform the test, and leave the environment in a clean state which does not affect the execution of the next test. If a test clutters the system in every execution, leaves it in a state that cannot be reset, this is a manual test candidate. Tests must also not clash with each other: multiple testers and pipelines must be able to execute the same test at the same time. If this is not possible, these groups of tests should be executed once a day in a pipeline [cron job](https://crontab.guru/#0_1-23_*_*_6-7), preferably outside of business hours.

Each test that must change the state of the environment has to be used as a setup-state-test and must ensure that the test environment is  able to be cleaned up before the next test.

It is preferred that UI tests do not repeat themselves as setup tests; API tests, Application Actions or DB seeding should be utilized wherever a UI test has to be used as a setup for another test.

**Setup vs Cleanup**: Setup (before all) is preferred over Cleanup (after all). Wherever possible, the test itself should take responsibility that it starts in a clean environment. However, as emphasized above, tests must not make it so that after their execution the next test cannot clean up the environment.

**Login**: Varieties of UI-login should only be used in their singular test cases. Any other tests that require login should use intrinsic API login and/or have a pre-configured test user.

**Test state setup**: It is encouraged that tests are isolated so that they do not rely on an entire setup before they can be executed. Example: if a group of tests may need a user to be created, a test user can be utilized to use these tests in isolation. On the other hand, the tests that setup the user should be independent and isolated.

**Modularity**: each test should be able to be run by itself, not relying on other tests to set up state for it. If this setup is required, it should be handled in beforeAll or beforeEach sections. A good way to test this is to run the test in isolation: `it.only()` , `fit()`, etc.).

```JavaScript
describe('..', function () {

  // setup (before/beforeEach) is preferred over cleanup (after/afterEach)

  before(function () {
    // login with UI once in an isolated test
    // for login here and all other tests, use a faster login method: use API, App Actions or DB seeding
  });

  beforeEach(function () {
    // setup additional state...
    // have one UI test to ensure this state can be achieved
    // however for the state set up here, utilize API, Application Actions or DB seeding; do not repeat UI tests
  });

    // test each test once with .only to ensure modularity
    it('..', function () {..});
    it('..', function () {..});
    it.only('..', function () {..});
    it('..', function () {..});

});

```

<br/><br/>

### References
[Stop using Page Objects and Start using App Actions](https://www.cypress.io/blog/2019/01/03/stop-using-page-objects-and-start-using-app-actions/)

[Cypress Docs: Organizing Tests, Logging In, Controlling State](https://docs.cypress.io/guides/references/best-practices.html#Organizing-Tests-Logging-In-Controlling-State)
