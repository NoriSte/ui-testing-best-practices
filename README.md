[✔]: assets/images/checkbox-small-blue.png

# UI Testing Best Practices

<h1 align="center">
  <img src="assets/images/banner-2.jpg" alt="UI testing Best Practices">
</h1>

<br/>

<div align="center">
  <img src="https://img.shields.io/badge/⚙%20Item%20count%20-%2024%20Best%20Practices-blue.svg" alt="24 items"> <img src="https://img.shields.io/badge/%F0%9F%93%85%20Last%20update%20-%20May%202021-green.svg" alt="Last update: May, 2021">
</div>

<br/>

[![NoriSte](/assets/images/twitter-s.png)](https://twitter.com/NoriSte/) **Follow me on Twitter!** [**@NoriSte**](https://twitter.com/NoriSte/)

<br/>

###### Built and maintained by our [Steering Committee](#steering-committee) and [Collaborators](#collaborators)

## Table of Contents

1.  [Testing strategies (4)](#1-testing-strategies)
2.  [Generic Best Practices (5)](#2-generic-best-practices)
3.  [Server Communication Testing (3)](#3-server-communication-testing)
4.  [Beginners (1)](#4-beginners)
5.  [Generic testing perks (1)](#5-generic-testing-perks)
6.  [Tools (1)](#6-tools)
7.  [Component Testing (3)](#7-component-testing)
8.  [Advanced (5)](#8-advanced)
9.  [Real Life Examples (1)](#9-real-life-examples)

<br/><br/>

# `1. Testing strategies`

## ![✔] 1.1 Component tests vs (UI) Integration tests vs E2E tests

**TL;DR:** Identifying the test types is the starting point to understand and master all the UI testing strategies, the tools, and the pro/cons of them. UI integration tests are the most effective ones (you are going to love them), E2E tests give you the highest confidence, and Component tests allow you to test the units of the UI in isolation.

**Otherwise:** You end up writing a lot of E2E tests without leveraging other simpler kind of tests. E2E tests are the most confident type of tests but even the hardest, slowest and most brittle ones.

🔗 [**Read More: Component vs (UI) Integration vs E2E tests**](/sections/testing-strategy/component-vs-integration-vs-e2e-testing.md)

<br/>

## ![✔] 1.2 In the beginning, avoid perfectionism

**TL;DR:** Software Testing is an amazing topic but a limited experience could make you fighting with a new enemy instead of relying on a new ally. Avoid, if you can, to test every complex user flows since the beginning of your UI testing journey. The simpler your first tests are, the sooner you get the advantages.

**Otherwise:** You create complex and hard to be debugged tests. This kind of tests slow down your work and do not have any kind of usefulness.

🔗 [**Read More: In the beginning, avoid perfectionism**](/sections/testing-strategy/avoid-perfectionism.md)

<br/>

## ![✔] 1.3 Choose a reference browser

**TL;DR:** Cross-browser testing is way overrated. It's an important topic and it's the first thing you can think while starting evaluating the right testing tool. Don't worry: start by splitting functional testing from visual testing, that's the first step to correctly evaluate the need for cross-browser support (and to choose the right testing tool, too). Visual testing can be integrated into every testing tool, thank services like Applitools and Percy.

**Otherwise:** You could choose the wrong testing tool based on the cross-browser support.

🔗 [**Read More: Choose a reference browser**](/sections/testing-strategy/choose-a-reference-browser.md)

<br/>

## ![✔] 1.4 Found a bug? Write the test, then fix it

**TL;DR:** A test is a good ally when you need to be sure that you are able to systematically reproducing a bug. A test allows you to speed up the fixing flow and to be 100% confident that the same bug is caught forever.

**Otherwise:** You could not identify correctly the bug and you can not be sure that the bug will not present again in the future.

🔗 [**Read More: Found a bug? Write the test, then fix it**](/sections/testing-strategy/write-test-then-fix-bug.md)

<br/>

# `2. Generic Best Practices`

## ![✔] 2.1 Await, don't sleep

**TL;DR:** When testing your UI, you define a sort of key points the app must pass through. Reaching these key
points is an asynchronous process because, almost 100% of the times, your UI does not update
synchronously. Those key points are called **deterministic events**, as known as something that you
know that must happen. You need to wait for these events to make your tests robust.

**Otherwise:** Sleeping the tests make your tests slow and brittle, it's one of the most common and biggest errors in UI testing.

🔗 [**Read More: Await, don't sleep**](/sections/generic-best-practices/await-dont-sleep.md)

<br/>

## ![✔] 2.2 Name your test files wisely

**TL;DR:** Lot of times you need to launch just a type of tests and it's super easy if you follow a
common pattern while naming your testing files.

**Otherwise:** You need to launch a long test suite just to have some of them run.

🔗 [**Read More: Name the test files
wisely**](/sections/generic-best-practices/name-test-files-wisely.md)

<br/>

## ![✔] 2.3 UI Tests Debugging Best Practices

**TL;DR:** Debugging a UI test could be really hard, especially if you use generic browser automation tools. Here is a list of simple rules that are at the base of the debugging process.

**Otherwise:** You are going to waste a lot of time without taming the exponential complexity of a UI test.

🔗 [**Read More: UI Tests Debugging Best Practices**](/sections/generic-best-practices/ui-tests-debugging-best-practices.md)

<br/>

## ![✔] 2.4 Reaching UI state for tests without using the UI

**TL;DR:** As a developer who wants to ensure quality, it is important to think about cost of tests vs the value they provide. Where reasonable, strive to not duplicate effort, and still get high value by considering alternatives for setting up state for a test.

🔗 [**Read More: Reaching UI state**](./sections/generic-best-practices/reaching-ui-state.md)

<br/>

## ![✔] 2.5 Use your testing tool as your primary development tool

**TL;DR:** Leveraging your testing tool to avoid manual tests is one of the biggest improvements you
could do to speed up your working flow. Testing tools are faster than you and the most modern ones include
some UI utilities that make easy to use them as a development tool.

**Otherwise:** You code the app the old way, losing a lot of time interacting manually with the UI itself.

🔗 [**Read More: Use your testing tool as your primary development tool**](/sections/generic-best-practices/use-your-testing-tool-as-your-primary-development-tool.md)

<br/><br/>

# `3. Server Communication Testing`

## ![✔] 3.1 Test the request and response payloads

**TL;DR:** The UI communicates continuously with the back-end, and usually every communication is critical. A bad request or a bad response could cause inconsistent data and inconsistent UI state. Remember that all the business is built around data and the user experience is scratched by every single UI failure. So, every single XHR request must be checked carefully. XHR request checks make your test more robust too, correct XHR management and testing are one of the most important aspects of a UI test.

**Otherwise:** You could miss some relevant communication inconsistencies and when you need to debug them, you are going to waste a lot of time because the test will not drive you directly to the issue.

🔗 [**Read More: Test the request and response payloads**](/sections/server-communication-testing/test-request-and-response-payload.md)

<br/>

## ![✔] 3.2 Test the server schema

**TL;DR:** A lot of times, the front-end application breaks because of a change in the back-end. Ask your back-end colleagues to allow you to export every schema that describes the back-end entities and the communication with the front-end. Some examples could be the GraphQL schema, the ElasticSearch mapping, a Postman configuration, etc. more in general, everything that can warn you that something changed in the back-end. Every back-end change could impact the front-end and you must discover it as soonest as possible. You can keep the schema checked with a simple [snapshot test](https://jestjs.io/docs/en/snapshot-testing).

**Otherwise:** You could miss some back-end change and your front-end application could break inadvertently.

<br/>

## ![✔] 3.3 Monitoring tests

**TL;DR:** The more the test suites are launched periodically, the more confident you are that everything works as expected. UI tests should be based on the user perspective but there are a lot of small tests that could give you a lot of immediate feedback without debugging the expected user flows. Monitoring small and taken-for-granted tech details helps you preventing bigger test failures.

**Otherwise:** You mix tech-details tests with the user-oriented ones.

🔗 [**Read More: Monitoring tests**](/sections/server-communication-testing/monitoring-tests.md)

<br/><br/>

# `4. Beginners`

## ![✔] 4.1 Approach the testing pyramid from the top!

**TL;DR:** Approaching the testing world could be inefficient and not satisfactory. You start writing some unit tests but you are left with a lot of doubts. UI Testing allows you to start with a high confidence since the very first day.

**Otherwise:** The wrong approach could condition the way you think about testing and could leave you with the false idea of testing the right way when the truth is you're testing nothing.

🔗 [**Read More: Approach the testing pyramid from the top!**](/sections/beginners/top-to-bottom-approach.md)

<br/><br/>

# `5. Generic testing perks`

## ![✔] 5.1 Software tests as a documentation tool

**TL;DR:** Tests are a good way to have a concise, code-coupled, and updated documentation. Good storytelling test descriptions could make the comprehension of a codebase or a new project very simple.

**Otherwise:** You rely on the code documentation or, worse, on the readability of the code to comprehend that the code does.

🔗 [**Read More: Software tests as a documentation tool**](/sections/testing-perks/tests-as-documentation.md)

<br/><br/>

# `6. Tools`

## ![✔] 6.1 Some UI testing problems and the Cypress way

**TL;DR:** Why is testing a web application so hard? Why generic browser automation tools do not fit well the UI/E2E testing needs? Why does Cypress outstand?

**Otherwise:** A generic features comparison is not enough to understand what are the main UI Testing pains and how Cypress removes them.

🔗 [**Read More: Some UI testing problems and the Cypress way**](/sections/tools/ui-testing-problems-cypress.md)

<br/><br/>

# `7. Component Testing`

Please note: the whole Component Testing world is changing rapidly, more contents will be added.

## ![✔] 7.1 [@NoriSte](https://github.com/NoriSte)'s approach: Testing a component with Cypress and Storybook

**TL;DR:** Components ar the building blocks of your app, testing them in isolation is important to discover, as soon as possible, iof there is something wrong with them.

**Otherwise:** UI Tests without lower-level tests do not allow you to understand the source of the problem.

🔗 [**Read More: Testing a component with Cypress and Storybook**](/sections/tools/cypress-and-storybook.md)

<br/>

## ![✔] 7.2 [@daedalius](https://github.com/daedalius)'s approach: Exposing components from Storybook separating stories from tests

**TL;DR:** You may expose the component reference from Storybook Story to test it whatever you wish in Cypress without breaking testing logic into pieces.

**Otherwise:** Splitted test logic and test data will make it difficult to read and support.

🔗 [**Read More: Cypress + Storybook. Keeping test scenario, data and component rendering in one place.**](/sections/tools/cypress-and-storybook-exposing-component-from-story.md)

<br/>

## ![✔] 7.3 Unit Testing React components with Cypress

**TL;DR:** Cypress v4.5.0 release allowed Unit Testing React components, an external tool like Storybook is not necessary anymore to test isolated components.

🔗 [**Read More: Unit Testing React components with Cypress.**](/sections/tools/cypress-react-component-test.md)
<br/> <br/>

# `8. Advanced`

## ![✔] 8.1 Test States

**TL;DR:** Tests should be repeatable, modular and should handle their own state setup. UI Tests should not be repeated in order to achieve state for another test.

🔗 [**Read More: Test States**](./sections/advanced/test-states.md)

<br/>

## ![✔] 8.2 Test Flake

**TL;DR:** Tests must produce consistent results every time. Repeatable pipeline execution results are the quorum.
If a test cannot produce reliable results, it reduces confidence in the tests and requires maintenance which reduces all value. In these cases it is best to manually test the functionality.

🔗 [**Read More: Test Flake**](./sections/advanced/test-flake.md)

<br/>

## ![✔] 8.3 Combinatorial Testing

**TL;DR:** Most software bugs and failures are caused by one or two parameters. Testing parameter combinations can provide more efficient fault detection than conventional methods. Combinatorial Testing is a proven method for more effective software testing at a lower cost.

🔗 [**Read More: Combinatorial Testing**](./sections/advanced/combinatorial-testing.md)

<br/>

## ![✔] 8.4 Performance Testing

**TL;DR:** Although this is a vast topic, Performance testing from a web development perspective can be simplified with modern tools and understanding. It is highly effective in ensuring user experience, satisfying non-functional requirements (NFRS), and detecting possible system-flake early on.

🔗 [**Read More: Performance Testing**](./sections/advanced/performance-testing.md)

<br/>

## ![✔] 8.5 Email Testing

**TL;DR:** Email testing is [critical for business success](https://www.industrialmarketer.com/why-email-testing-is-critical-for-email-marketing-success/). Modern services not only allow automated email testing but also provide a stateless, scalable solution while testing SaaS applications.

🔗 [**Read More: Email Testing**](./sections/advanced/email-testing.md)

<br/><br/>

# `9. Real Life Examples`

## ![✔] 9.1 Siemens - Test the front-end with the integration tests, the back-end with the E2E ones - in reference to [Component vs Integration vs E2e Testing](./sections/testing-strategy/component-vs-integration-vs-e2e-testing.md)

**TL;DR:** UI tests with a stubbed server are reliable and faster compared to full E2E tests. Full E2E tests are not always necessary to ensure front-end quality. We can instead have high confidence in front-end quality by using lower-cost UI integration tests and saving higher cost E2E tests for the back-end.

**Otherwise:** You waste time and resources with slow and brittle E2E tests while you can get a lot of confidence with a lot of UI integrations tests.

🔗 [**Read More: Test the front-end with the integration tests, the back-end with the E2E ones**](./sections/real-life-examples/test-front-end-with-integration-back-end-with-e2e.md)

<br/> <br/>


## Steering Committee

Meet the steering committee members - the people who work together to provide guidance and future direction to the project.

<img align="left" width="100" height="100" src="assets/images/members/noriste.png">

[Stefano Magni](https://github.com/NoriSte)
<a href="https://twitter.com/NoriSte"><img src="assets/images/twitter-s.png" width="16" height="16"></img></a>
<a href="https://www.linkedin.com/in/noriste/"><img src="assets/images/linkedin.png" width="16" height="16"></img></a>
<a href="https://stackoverflow.com/users/story/700707"><img src="assets/images/stackoverflow.png" width="16" height="16"></img></a>

A positive-minded front-end developer and Cypress Ambassador. He's a passion for good UIs, automation, testing and teaching. He's developed every kind of interface: webapps, mobile apps, smartTV apps and games.

<br/>

## Collaborators

Thank you to all our collaborators! 🙏

Our collaborators are members who are contributing to the repository on a reguar basis, through suggesting new best practices, triaging issues, reviewing pull requests and more. If you are interested in helping us guide thousands of people to craft better UI tests.

<br/>

|              <a href="https://github.com/muratkeremozcan" target="_blank"><img src="https://avatars.githubusercontent.com/muratkeremozcan" width="75" height="75"></a>               |
| :----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------: |
| [Murat Ozcan](https://github.com/muratkeremozcan) <a href="https://www.linkedin.com/in/murat-ozcan-3489898/"><img src="assets/images/linkedin.png" width="16" height="16"></img></a> |

<br/>

## Thank You Notes

We appreciate any contribution, from a single word fix to a new best practice. Below is a list of everyone who contributed to this project. A 🌻 marks a successful pull request and a ⭐ marks an approved new best practice.

### Stars

An approved new best practice Be the first to collect a ⭐, contribute to this repository 😁

⭐ [Murat Ozcan](https://github.com/muratkeremozcan)
⭐ [Dmitriy Tishin](https://github.com/daedalius)

### Flowers

A successfull PR gives you a 🌻, be the first to collect it.

🌻 [Anoop Kumar Gupta](https://github.com/anoop-gupt)
🌻 [Ferdinando Santacroce](https://github.com/jesuswasrasta)
🌻 [Luca Piazzoni](https://github.com/bioz87)
🌻 [Luca Previtali](https://www.linkedin.com/in/previtaliluca/)
🌻 [Luca Previtali](https://www.linkedin.com/in/previtaliluca/)
🌻 [Filip Hric](https://github.com/filiphric)

<br/><br/><br/>

This repository is inspired by the [nodebestpractices](https://github.com/i0natan/nodebestpractices) one, thank you [Yoni](https://github.com/i0natan) and the whole [steering team](https://github.com/i0natan/nodebestpractices#steering-committee) to keep it updated and to allow the creation of this repository.

<br/><br/><br/>
