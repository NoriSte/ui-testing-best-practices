# Hasura E2E tests chronicles, February 2023

<br/><br/>

### One Paragraph Explainer

Despite the efforts put in trying to debug/save/update the high-failing E2E tests, Hasura decided to wipe out almost all the previous E2E tests. There were a lot of problems, and most of them were a consequence of a CI misconfiguration.

<br/><br/>

I joined Hasura in May 2022, and one of my first tasks was to fix the E2E tests of the Hasura Console, main Hasura's front-end application.

The main problems were: they were slow, and they were flaky. Then, by digging into the topic, there was more to say, more to decide, more to fix, and more to do. Let me elaborate a bit more:

1.  The E2E tests were **slow**: a lot of `cy.wait(10000)` (yes, ten seconds) everywhere.
2.  The E2E tests were **flaky**: a couple of months before I joined Hasura, the whole company complained about the CI jobs' flakiness, preventing the teams from merging their PRs. The problem has been workaround'ed by enabling Buildkite (our CI tool) to retry the E2E test job.
3.  The E2E tests were **cryptic**. Understanding the E2E tests (and fixing/refactoring them) was hard because they were long, terse, and had many abstractions.
4.  Debugging the E2E tests was challenging because of the many different modes the Hasura Console can launch.
5.  The server team used the E2E tests also to **test the server**.
6.  Cypress crashed because the Console was too resource-demanding.

Let's go through every single topic, one by one.

_Photo by <a href="https://unsplash.com/@claybanks?utm_source=unsplash&utm_medium=referral&utm_content=creditCopyText">Clay Banks</a> on <a href="https://unsplash.com/photos/pNEmlb1CMZM?utm_source=unsplash&utm_medium=referral&utm_content=creditCopyText">Unsplash</a>_


## The E2E tests were slow.

In the [Await, don't sleep](/sections/generic-best-practices/await-dont-sleep.md) chapter, I shared why having a fixed amount of waiting/sleep is terrible in the tests, following the gist of it.

Fixed waitings are one of the most common causes of slow tests. The tests get slow by always waiting for 10 seconds when an XHR request is happening, even if it usually takes less than 1 second. And for the rare cases where the XHR request takes more than 10 seconds (imagine a cold server start), the tests will fail.

Instead, the **test should wait for something that happens deterministically **(the XHR request, an element to appear, etc.). Cypress eases avoiding sleep with its retry-ability (see [the docs here](https://docs.cypress.io/guides/core-concepts/retry-ability#Commands-vs-assertions)) and with built-in timeouts, such as:

-   up to 60 seconds for a page to trigger the load event when visited
-   up to 4 seconds for an element to appear before interacting with it
-   up to 5 seconds for an XHR request to start and up to 30 seconds to end
-   etc.

Another **critical problem** of fixed waitings: **the developer cannot understand if the test's creator initially wanted to wait for an XHR request, for an element to appear, or for an animation**, slowing down the debugging and refactoring processes.

Guess what? Replacing the sleep with proper "wait for the XHR request to happen" was almost impossible because of a series of problems:

1.  The Hasura Console comprises legacy and more modern code with no central server-state management. Since different code ages coexist on the same page, other code parts fetch the same data differently. This results in many useless requests and the **impossibility of predicting the order of the XHR requests**. No order, no party, because Cypress will fail to wait for these crazy requests with many false negatives.
2.  The Hasura Console can use different servers based on how the consumers use it. The main difference is that the Console can run in server mode (talking to the Hasura GraphQL Engine--HGE-- server) or in CLI mode (talking to a locally running CLI server that, in turn, talks with the HGE server). Different servers mean different URLs, different ports, and different APIs, making it **almost impossible to intercept the XHR requests**.

So? I see two leading possible solutions:

1.  The one to use most of the time: waiting for something that reflects the fact that the XHR request happened (ex., the success notification, something appearing in the UI) with a longer delay (remember, Cypress waits up to 30 seconds by default for an XHR request and there is a reason), and a comment for the readers that the long delay aims to replace the unfeasibility of intercepting XHR requests.
2.  Intercepting two possible requests instead of one. Following is a snippet coming from our codebase

```ts
cy.intercept('POST', 'http://localhost:8080/v1/metadata', req => {
  if (JSON.stringify(req.body).includes('create_action')) {
    req.alias = 'createAction';
  }
  req.continue();
});

cy.intercept('POST', 'http://localhost:9693/apis/migrate', req => {
  if (JSON.stringify(req.body).includes('create_action')) {
    req.alias = 'createAction';
  }
});

cy.getBySel('create-action-btn').click();

cy.wait('@createAction').then(interception => {
  checkMetadataPayload(interception, { name: 'Action payload' });
```

3\. The one I do not suggest: waiting for some requests to happen before proceeding with the test. Why do not I recommend it? Because of the edge cases to care about and the resulting complex code.

<details>
  <summary>Following is an example (get ready for something shocking).</summary>

```ts
/**
 * Wait for a bunch of requests to be settled before proceeding with the test.
 *
 * Alternatively, https://github.com/bahmutov/cypress-network-idle could be used
 *
 * This is a workaround for "element is 'detached' from the DOM" Cypress' error (see the issue
 * linked below). Since the UI gets re-rendered because of the requests, this utility ensures that
 * all the requests parallelly made by the UI are settled before proceeding with the test. Hence, it
 * ensure the UI won't re-render during the next interaction.
 *
 * What are the requests that must be awaited? By looking at the Cypress Test Runner, they are the
 * following, made parallelly or in a rapid series.
 * 1. export_metadata
 * 2. export_metadata
 * 3. export_metadata
 * 4. test_webhook_transform
 * 5. test_webhook_transform
 * 6. test_webhook_transform
 * 7. test_webhook_transform
 * At the moment of writing, I'm not sure the number of requests are fixed or not. If they are fixed,
 * using the cy.intercept `times` options would result in a more expressive and less convoluted code.
 *
 * To give you an overall idea, this is a timeline of the requests
 *
 *         all requests start                             all requests end
 *         |                 |                            |               |
 * |--🚦🔴--1--2--3--4--5--6--7----------------------------1--2--3--4--5--6-7--🚦🟢--|
 *
 *
 * ATTENTION: Despite the defensive approach and the flakiness-removal purpose, this function could
 * introduced even more flakiness because of its empiric approach. In case of failures, it must be
 * carefully evaluated when/if keeping it or thinking about a better approach.
 * In general, this solution does not scale, it should not be spread among the tests.
 *
 * @see https://github.com/cypress-io/cypress/issues/7306
 * @see https://glebbahmutov.com/blog/detached/
 * @see https://github.com/bahmutov/cypress-network-idle
 */

import 'cypress-wait-until';

export function waitForPostCreationRequests() {
  let waitCompleted = false;

  cy.log('*--- All requests must be settled*');

  const pendingRequests = new Map();
  cy.intercept('POST', 'http://localhost:8080/v1/metadata', req => {
    if (waitCompleted) return;

    Cypress.log({ message: '*--- Request pending*' });

    pendingRequests.set(req, true);

    req.continue(() => {
      Cypress.log({ message: '*--- Request settled*' });

      pendingRequests.delete(req);
    });
  });

  Cypress.log({ message: '*--- Waiting for the first request to start*' });

  // Check if at least one request has been caught. This check must protect from the following case
  //
  //            check          requests start           test failure, the requests got the UI re-rendered
  //            |              |                        |
  // |--🚦🔴----⚠️---🚦🟢-------1-2-3-4-5-6-7-1----------💥
  //
  // where checking that "there are no pending requests" falls in the false positive case where
  // there are no pending requests because no one started at all.
  //
  // The check runs every millisecond to be 100% sure that no request can escape (ex. because of a
  // super fast server). A false-negative case represented here
  //
  //         requests start requests end   check              check               test failure, no first request caught
  //         |            | |           |  |                  |                   |
  // |--🚦🔴--1-2-3-4-5-6-7-1-2-3-4-5-6-7--⚠️------------------⚠️------------------💥
  cy.waitUntil(() => pendingRequests.size > 0, {
    timeout: 5000, // 5 seconds is the default Cypress wait for a request to start
    interval: 1,
    errorMsg: 'No first request caught',
  });

  Cypress.log({ message: '*--- Waiting for all the requests to start*' });

  // Let pass some time to collect all the requests. Otherwise, it could detect that the first
  // request complete and go on with the test, even if another one will be performed in a while.
  //
  // This fixed wait protects from the following timeline
  //
  //           1st request start     first request end       other requests start   test failure, the requests got the UI re-rendered
  //           |                     |                       |                      |
  // |--🚦🔴---1---------------------1----🚦🟢----------------2-3-4-5-6-7-1----------💥
  //
  // Obviously, it is an empiric waiting, that also slows down the test.
  cy.wait(500);

  Cypress.log({ message: '*--- Waiting for all the requests to be settled*' });

  cy.waitUntil(() => pendingRequests.size === 0, {
    timeout: 30000, // 30 seconds is the default Cypress wait for the request to complete
    errorMsg: 'Some requests are not settled yet',
  }).then(() => {
    waitCompleted = true;
  });
}
```
</details>

I think it's a pity not to intercept the XHR requests because it could act as a life-saver that, when something is not working, allows you to immediately understand if it's the front-end fault (because of a wrong request payload) or a back-end fault (because of an incorrect response payload, status code, etc.). I can live with it, anyway 😊

## The E2E tests were flaky

Flakiness is always a red alert. You should monitor your tests and always fix the E2E tests. If you can't fix them now, skip them, and explain why. **Flaky tests undermine the working flow**, undermine the confidence and trust in the E2E tests, make everyone hate them, and add more friction than the one they are supposed to remove. **No tests are way better than flaky tests**.

On the Hasura Console, not only has the flakiness not been fixed but the significant, involuntarily error that has been made is enabling Buildkite retry mechanisms in case of E2E test failures... without checking what was happening under the hood. Why? Because Buildkite retries the failing E2E tests without passing a different id to the Cypress CLI. As a result, Cypress does not run any test in the retry because it knows that a test run with the same id already happened! **No test run means no failing tests, which means CI goes green**.


![Buildkite showing a failure, retrying the tests, then going green because Cypress does nothing](/assets/images/hasura-chronicles/buildkite-retry.jpg)
_Buildkite showing a failure, retrying the tests, then going green because Cypress does nothing._



Can you spot the terrible problem? CI is going green for failing tests, with the developers thinking that the tests were succeeding. The result is not only false confidence but also **the E2E tests were less and less aligned with the application** (highlighting another problem, no one was running the tests locally while working...).

Another thing to consider: running all the E2E tests, then re-starting Cypress to re-run them means that every PR run the complete test suite, paying for AWS running them and for the Cypress Cloud dashboard, paying the time cost of waiting for the completion, with no value at all!


![The Buildkite jobs showing the E2E tests require almost 10 minutes](/assets/images/hasura-chronicles/buildkite-test-duration.png)
_The Buildkite jobs showing the E2E tests require almost 10 minutes._


## The E2E tests were cryptic

Test code must be 100x simpler than application code. It must be a clear goal when writing tests, not a by-product of writing a program (the test) that's 100x simpler by definition.

Anyway, this topic is long and deserves a dedicated chapter. You can find it here: [Keep abstraction low to ease debugging the tests](/sections/generic-best-practices/test-code-with-debugging-in-mind.md).

## Debugging the E2E tests was challenging

This is the result of multiple factors:

1.  The E2E tests are hard to read (look at the previous chapter, "The E2E tests were cryptic").
2.  The Console runs in different modes (see the "The E2E tests were slow." chapter), which is not easy to detect at first glance.
3.  The tests are coupled, and test B needs to run after test A. This is another exciting and lengthy debate. I split it into a dedicated chapter, too. See "[One long E2E test or small, independent ones?](/sections/testing-strategy/small-tests-or-long-ones.md)".

But please keep in mind that **debugging E2E tests is always hard**. The only game changer I know (and suggest) is [Replay](https://www.replay.io/), which now [allows recording browser tests](https://docs.replay.io/recording-browser-tests-(beta)).

## The server team used the E2E tests also to test the server

On the server, the situation was even worse. There were no integration tests that allowed us to be sure the server always respected the contract. As a result, the Console's E2E tests were also used to check that "if the Console works, the server did not break any API".

Apart from the false confidence our Buildkite misconfiguration gives, this prevents the Console tests from scaling. It's good to have some E2E tests, but since they are super slow and sometimes flaky, we need to keep them at a minimal number. Most of the testing focus should be on the front-end and back-end sides, but independently from each other.

You can find all the details and the rationales in the dedicated "[Decouple the back-end and front-end test through Contract Testing](/sections/real-life-examples/hasura-contract-tests.md)" chapter, which is the proposal I made internally to split the Console and server tests.

## Summary and the future

So, in 2022:

1.  We started refactoring some tests (but never completed them).
2.  We started the Contract Testing proposal (but have yet to push it seriously).
3.  We tracked and skipped all the flaky tests, willing to fix the CI misconfiguration (we never fixed it because of other CI problems).
4.  We enabled Slack alerts from Cypress to quickly identify other flaky tests, skip them or fix them.
5.  We started using proper Contract Testing methodology on a small portion of features.

 Then, an internal Working Group was created to fix our numerous CI problems, and the E2E tests were one of the most problematic ones. And since they are slow, they are flaky, they add no value, they are outdated...

... We opted for the hardest but best decision: to eradicate the E2E tests!


![The description of the big PR removing the E2E tests](/assets/images/hasura-chronicles/wipeout-pr.png)
_The description of the big PR removing the E2E tests._


Let's start with a clean slate and think about the future! Some points that will positively impact the E2E tests:

1.  We will consider **Playwright instead of Cypress** due to the speed and fewer resources required. The first round of using it went well. Now we need to check if it fulfills all our needs.
2.  Refactors are happening across some features, pushing **more tests to the interaction level (Storybook)** level instead of the full-app-in-browser level.
3.  The recent migration to **Nx reduced the size of the Console by 70%**, which means a faster startup and faster browser tests.
4.  We have too many types/modes the Console can run, some of them will be removed from a Product perspective.
5.  We (the frontenders of the Platform Team) will dedicate time **helping and mentoring** the frontenders of the Feature teams to write more scalable tests.

But there is one real game changer: **the server will auto-generate TypeScript types** for all the server objects and APIs! That means that on the Console, we will always be using the latest and correct types, guaranteed by the server, and **we can start trusting whatever comes from the server**!

Ensuring the server respects its part of the contract means we will not need so many E2E tests, but server-free tests against the server types will be our safety net! This changes everything in terms of the need for a lot of E2E tests!!!

Stay tuned. Maybe next year, we will share how we are doing in terms of front-end testing ❤️


<br/><br/>

*Crossposted by [NoriSte](https://github.com/NoriSte) on [dev.to](https://dev.to/noriste/hasura-e2e-tests-chronicles-february-2023-24ki) and on the [Hasura blog](https://hasura.io/blog/why-we-deleted-95-of-our-e2e-tests/).*
