# Decouple the back-end and front-end test through Contract Testing

<br/><br/>

### One Paragraph Explainer

E2E tests do not scale well for hig applications, with thousands of user flows. Things get even worse if the UI E2E tests are used to test the server too because the result is hundred of slow and coupled tests. In Hasura, an internal working group proposed to decouple the back-end and front-end tests.

<br/><br/>

## Context

I (a Front-end engineer, keep it in mind) recently championed an initiative aimed at reducing Hasura's CI pipeline duration and the dependence on not-scaling-well E2E tests.

The problem is that **the Server team defensively uses the Hasura Console's** (the main UI of the company) **E2E tests** to check for regressions. If the Console works as expected, the Server works as expected as a by-product.
The CI pipelines must build both the Server and the Console and then launch the Console's E2E tests. This happens for every PR, both the Server-related ones and the Console-related ones.

![A graph showing the process of building both the parts of the system to test them](/assets/images/hasura-contract-tests/coupled-tests.png)

Why is it a problem? Because the Console takes 10 minutes to build and 15 minutes to run the tests (spread across eight processes at the moment). Waiting **25 minutes to ensure that a bunch of Server endpoints correctly does not scale**.

How can we reduce this problem?
1. By relying on more low-level tests on the Server side
2. By using a series of requests performed against the Server, waiting for some exact responses. The series of requests and responses made the "contract" between the Console and the Server. Testing this contract would partially replace the E2E tests.

![A graph showing the both the parts of the system tested individually](/assets/images/hasura-contract-tests/decoupled-tests.png)


You can find all the info/answers I shared internally below. Please note that the questions below refer to the ones everyone has to answer before proposing a topic to the internal Architectural Working Group.

---

## Step 1: the proposal to the Architectural Working Group

Questions that [foster dialogue](https://www.kitchensoap.com/2017/03/06/book-suggestion-dialogue-the-art-of-thinking-together/)

_What problem are you solving?_
1. **Reducing the number of E2E tests** by introducing Integration tests for the Console
2. **Speeding up the CI** pipelines by leveraging faster tests
3. Avoiding the Server requiring to build the Console and launching its tests to know if an API works or not
4. Avoiding the Console devs to have the Server working locally or in the cloud
5. Detecting regressions upfront and allowing to notice what is the component of the equation that introduced the regression (the frontend or the backend)

_Why are you solving this problem?_
1. Because **relying on E2E tests does not scale**. Backend and frontend could have slightly different testing pyramids/trophies, but in both cases, E2E tests always stay at the top: high confidence at the cost of flaky and extremely slow tests
2. Because **the feedback loop is too long**, building the Console + launching its tests take at least 25 minutes (thanks to CI parallelization, otherwise it's even worse)

_Is it possible I don't even have to solve this problem?_
1. No. Regressions on the Server cannot be found out after a 30-min CI
2. No (2), because the Console's UI is not fully covered in terms of E2E tests and will never be. Otherwise, we end up having 5-hours CIs, not 30-mins ones
3. Wait... Wasn't you, Stefano, fixing the Console's E2E tests? Why do you want to get rid of them? Because "fixing" means the following points, but they do not resolve the root cause of the problem of using E2E tests
  - Removing the source of flakiness entirely but it's not quite possible because of the large number subjects involved (FE+BE+DB)
  - Speeding up the tests, but we cannot expect more than a 30%-50% improvement
  - Making them helpful in case of failures
4. The Console will be migrated to [Nx](https://nx.dev/), and in the future, the result of E2E tests will be cached, why caring about their slowness? Because Nx caches the E2E tests based on the parts of the UI that change. If the server change, Nx cannot detect it, and they won't run the E2E tests, resulting in server-related regressions going directly to production

_How would you solve your immediate problem without adding anything new?_
By removing some E2E tests and writing Console's tests against static fixtures. As a result, the Server loses confidence because it's coupled with the Console's E2E tests.

_Write what it is about the current stack that makes solving the problem prohibitively expensive and difficult._
Checking for server API regressions means building the Console (10 minutes), launching the Console's E2E tests (15 minutes, thanks to CI parallelization), and we have only ~200 E2E tests out of the thousands of things the users can do with the Console.

_What are other potential solutions for the problem?_
The server APIs could also be tested in isolation, but… I imagine that the problem here is the application state. The Console's E2E tests are convenient from this point of view because usually they
- Create the entity
- Modify the entity
- Delete the entity
Hence the "modify the entity" API is called after the "create the entity" one, creating the application state needed for the former API to work.
At the same time, I think that a proper testing suite could reduce the need for this "APIs called from an external component against a working server" but this is out of my frontend-only experience.

_How can you build confidence in a specific solution approach?_
Confidence comes from the fact that we would test the same things as before, but in a different way

_What is the CON of the approach?_
The Contracts and the static fixtures must be kept updated. The Contracts will be generated manually, while the static fixtures used by the UI should be typed

_What are the tradeoffs and consequences of the solution?_ (answer by one of the Server folks)
- Currently, the Server gets a lot of release confidence from the console E2E tests. Implementing this strategy would mean more work by the server team to preserve that level of confidence.
- The majority of coverage for the backend currently centres around consumption of the GQL API ("if I send this GQL, I get this JSON/YAML") - a similar approach would work for the console APIs ("if I send this JSON to HGE, I expect to receive this JSON"), but this would require some upfront work.
- There could be a drift between the actual server behaviour and the behaviour stated by the mocks. This can be mitigated in a number of ways, though.
- Potentially, the space between FE/BE engineers will grow further, though vertical team structures should help a lot here.
All in all, the migration away from E2E tests for the Console would require a reasonable amount of test suite effort for BE teams. However, the benefit will most likely be a substantial improvement in CI times, and there's no reason we can't, say, treat the console team mocks as integration tests for the API.

## Step 2: the implementation of the automatic recorder

You can find the code [here](https://github.com/hasura/graphql-engine/blob/576417408709b1f9eb981a2fd3106ef523955aee/console/cypress/support/contractIntercept/contractIntercept.ts).

## Step 3: Closing thoughts

Compared to the initial proposal
1. The contract is not automatically recorded. The developer must adapt the E2E test following [this guide](https://github.com/hasura/graphql-engine/blob/576417408709b1f9eb981a2fd3106ef523955aee/console/src/docs/dev/testing/6-export-contract-from-e2e-test.stories.mdx) to record the contract
2. I’m going to remove the E2E tests that are almost duplicates (ex. creating a query action or a mutation does not differ, I’ll remove one of them)
3. For other E2E tests, I’ll share the contract before contextually
4. All the features that rely on the Console directly running SQL queries will rely on E2E tests, but only for the happy path
5. The frontenders won’t be able to automatically export the contract and use it for integration testing (read [here](https://stackoverflow.com/questions/68455650/cannot-make-cypress-and-pact-work-together/68492932#68492932) and [here](https://pactflow.io/blog/a-disastrous-tale-of-ui-testing-with-pact/)). Anyway, this is not a big problem because it’s strictly related to migrating the E2E tests to integration ones. When we are creating new features, the integration tests are smaller (that also means less requests), and we should have the typed mocks from the Storybook stories of the involved components that could also be imported in the Cypress tests.
6. The exported JSONs do not leverage [the power of Pact](https://github.com/pact-foundation/pact-specification/tree/version-3) (ex. cannot include special operators that allow for instance “to test that the id sent in the 3rd response is the same as the one sent in the 5th response”)

<br/><br/>

*Crossposted by [NoriSte](https://github.com/NoriSte) on [dev.to](https://dev.to/noriste/decouple-the-back-end-and-front-end-test-through-contract-testing-112k).*
