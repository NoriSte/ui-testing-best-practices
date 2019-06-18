# Name the test files wisely

<br/><br/>

### One Paragraph Explainer

You can write a lot of different UI tests and it's a good habit to have a common way of naming the
test files.

It's useful because often you need to run just a type of tests, the situations could be:
- during the development process, you need to run just some of them
  - you're changing some related components and you need to check that the generated markup does not change
  - you're changing a global CSS rule and you need to run only the visual tests
  - you're changing an app flow and you need to run the whole app integration tests
- your DevOps colleague needs to be sure that everything is up and running and the easiest way to do
  that is launching just the E2E tests
- your building pipeline needs to run just the integration and E2E tests
- your monitoring pipeline needs a script to launch the E2E and monitoring tests

If you name your tests wisely, it will be really easy to launch just some kind of them.

Cypress:
```bash
cypress run --spec \"cypress/integration/**/*.e2e.*\"
```

Jest:
```bash
jest --testPathPattern=e2e\\.*$
```

<br>

A global way to name the test files does not exist, a suggestion could be to name them with:
- the subject you are testing
- the kind of test (`integration`, `e2e`, `monitoring`, `component`, etc.)
- the test suffix of choice (`test`, `spec`, etc.)
- the file extension (`.js`, `.ts`, `.jsx`, `.tsx`, etc.)

all of them separated by a period.

Some examples could be
- `authentication.e2e.test.ts`
- `authentication.integration.test.ts`
- `site.monitoring.test.js`
- `login.component.test.tsx`
etc.
