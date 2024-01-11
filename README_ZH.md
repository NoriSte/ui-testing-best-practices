[✔]: assets/images/checkbox-small-blue.png

# UI 测试最佳实践

<h1 align="center">
  <img src="assets/images/banner-2.jpg" alt="UI testing Best Practices">
</h1>

<br/>

<div align="center">
  <img src="https://img.shields.io/badge/⚙%20Item%20count%20-%2026%20Chapters-blue.svg" alt="26 items"> <img src="https://img.shields.io/badge/%F0%9F%93%85%20Last%20update%20-%20Jun%202023-green.svg" alt="Last update: June, 2023">
</div>

<br/>

**在 Twitter 上关注我们！**:

- [**@NoriSte**](https://twitter.com/NoriSte/)
- [**@MuratKeremOzcan**](https://twitter.com/MuratKeremOzcan/)

<br/>

###### 由我们的[指导委员会]（#steering-committee）和[合作者]（#collaborators）构建和维护

## 目录

1. [测试策略 (5)](#1-testing-strategies)
2. [通用最佳实践 (6)](#2-generic-best-practices)
3. [服务器通信测试 (3)](#3-server-communication-testing)
4. [初学者 (1)](#4-beginners)
5. [通用测试的优势 (1)](#5-generic-testing-perks)
6. [工具 (2)](#6-tools)
7. [高级 (5)](#7-advanced)
8. [真实案例 (2)](#8-real-life-examples)
9. [过时章节 (3)](#9-obsolete-chapters)

<br/><br/>

# `1. 测试策略`

## ![✔] 1.1 组件测试 vs 集成测试 vs E2E 测试

**简而言之：** 辨识测试类型是理解和掌握所有 UI 测试策略、工具以及它们的利弊的起点。UI 集成测试是最有效的（你会喜欢上它们的），E2E 测试提供最高的信心，而组件测试则允许你独立测试 UI 的各个单元。

**或者说：** 否则，你可能会陷入过多编写 E2E 测试的困境，而忽略其他更简单的测试类型。E2E 测试是最为可靠的测试类型，但同时也是最难、最慢且最脆弱的一种。

🔗 [**阅读更多：组件测试 vs (UI) 集成测试 vs E2E 测试**](/sections/testing-strategy/component-vs-integration-vs-e2e-testing.zh.md)

<br/>

## ![✔] 1.2 在开始阶段，避免追求完美主义

**简而言之：** 软件测试是一个令人惊叹的话题，但有限的经验可能使您陷入与新敌人的斗争，而不是依赖新盟友。如果可能的话，在 UI 测试旅程的初期避免测试每个复杂的用户流程。您的第一个测试越简单，您越早获得优势。

**或者说：** 您将创建复杂且难以调试的测试。这种类型的测试会拖慢您的工作，而且毫无用处。

🔗 [**阅读更多：在开始阶段，避免追求完美主义**](/sections/testing-strategy/avoid-perfectionism.zh.md)

<br/>

## ![✔] 1.3 选择一个参考浏览器

**简而言之：** 跨浏览器测试被高估了。虽然这是一个重要的主题，也是在开始评估合适的测试工具时首先考虑的事项，但不必过于担心。首先，要将功能测试与视觉测试分开，这是正确评估是否需要跨浏览器支持的第一步，也是选择合适的测试工具的关键。视觉测试可以集成到每个测试工具中，这得益于诸如 Applitools 和 Percy 这样的服务。

**或者说：** 基于跨浏览器支持做出选择，可能导致选择错误的测试工具。

🔗 [**阅读更多：选择一个参考浏览器**](/sections/testing-strategy/choose-a-reference-browser.zh.md)

<br/>

## ![✔] 1.4 发现了 bug？先编写测试，然后再着手修复

**简而言之：** 在你需要确保能够有系统性地重现某个程序漏洞时，测试是一个极佳的助手。测试可以加速修复流程，同时让你百分之百确信同样的漏洞永远都能被捕捉到。

**或者说：** 如果你不能正确地辨别漏洞，那么你无法确定这个漏洞将来是否还会再次出现。

🔗 [**阅读更多：发现了 bug？先编写测试，然后再着手修复**](/sections/testing-strategy/write-test-then-fix-bug.zh.md)

<br/>

## ![✔] 1.5 单个长的端到端测试还是多个小的独立测试？

**简而言之：** 在处理端到端测试及其困难时，选择进行一次长时间的测试还是选择许多小而独立的测试并非显而易见。这两种解决方案都有各自的优劣，这源于端到端测试的内在复杂性，其中涉及真实后端和真实数据。

**或者说：** 你可能会创建难以维护的端到端测试。

🔗 [**阅读更多：单个长的端到端测试还是多个小的独立测试？**](/sections/testing-strategy/small-tests-or-long-ones.zh.md)

<br/><br/>

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

<br/>

## ![✔] 2.6 Keep abstraction low to ease debugging the tests

**TL;DR:** Tests should be written with readability and debuggability in mind. Abstraction may be good in some instances, but it always incurs a cost in debuggability and therefore sometimes may not be worth it. This is especially important for UI tests; consequent of the complex stack, it can get harder to understand the real source of failures. Reducing abstraction for the sake of easier debugging is key for future proofing the test code.

**Otherwise:** There is a balance between abstraction and debuggability; the higher the abstraction, the harder it is going to be to debug the tests in the future.

🔗 [**Read More: Keep abstraction low to ease debugging the tests**](/sections/generic-best-practices/test-code-with-debugging-in-mind.md)

<br/><br/>

# `3. Server Communication Testing`

## ![✔] 3.1 Test the request and response payloads

**TL;DR:** The UI communicates continuously with the back-end, and usually every communication is critical. A bad request or a bad response could cause inconsistent data and inconsistent UI state. Remember that all the business is built around data and the user experience is scratched by every single UI failure. So, every single XHR request must be checked carefully. XHR request checks make your test more robust too, correct XHR management and testing are one of the most important aspects of a UI test.

**Otherwise:** You could miss some relevant communication inconsistencies and when you need to debug them, you are going to waste a lot of time because the test will not drive you directly to the issue.

🔗 [**Read More: Test the request and response payloads**](/sections/server-communication-testing/test-request-and-response-payload.md)

<br/>

## ![✔] 3.2 Test the server schema

**TL;DR:** A lot of times, the front-end application breaks because of a change in the back-end. Ask your back-end colleagues to allow you to export every schema that describes the back-end entities and the communication with the front-end. Some examples could be the GraphQL schema, the TypeScript types, the ElasticSearch mapping, the Pact contract, a Postman configuration etc. more in general, everything that can warn you that something changed in the back-end. Every back-end change could impact the front-end and you must discover it as soonest as possible.

**Otherwise:** You could miss some back-end change and your front-end application could break inadvertently.

<br/>

## ![✔] 3.3 Monitoring tests

**TL;DR:** The more the test suites are launched periodically, the more confident you are that everything works as expected. UI tests should be based on the user perspective but there are a lot of small tests that could give you a lot of immediate feedback without debugging the expected user flows. Monitoring small and taken-for-granted tech details helps you preventing bigger test failures.

**Otherwise:** You mix tech-details tests with the user-oriented ones.

🔗 [**Read More: Monitoring tests**](/sections/server-communication-testing/monitoring-tests.md)

<br/><br/>

# `4. Beginners`

## ![✔] 4.1 Approach the testing pyramid from the top

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

## ![✔] 6.2 Visual Regression Testing

**TL;DR:** Visual regression tests hard and why we should rely on premium services.

**Otherwise:** Another continuous chore for regressions we do not care about. Possibility of missing out visual differences.

🔗 [**Read More: Visual Regression Testing**](/sections/tools/visual-regression-testing.md)

<br/><br/>

# `7. Advanced`

## ![✔] 7.1 Test States

**TL;DR:** Tests should be repeatable, modular and should handle their own state setup. UI Tests should not be repeated in order to achieve state for another test.

🔗 [**Read More: Test States**](./sections/advanced/test-states.md)

<br/>

## ![✔] 7.2 Test Flake

**TL;DR:** Tests must produce consistent results every time. Repeatable pipeline execution results are the quorum.
If a test cannot produce reliable results, it reduces confidence in the tests and requires maintenance which reduces all value. In these cases it is best to manually test the functionality.

🔗 [**Read More: Test Flake**](./sections/advanced/test-flake.md)

<br/>

## ![✔] 7.3 Combinatorial Testing

**TL;DR:** Most software bugs and failures are caused by one or two parameters. Testing parameter combinations can provide more efficient fault detection than conventional methods. Combinatorial Testing is a proven method for more effective software testing at a lower cost.

🔗 [**Read More: Combinatorial Testing**](./sections/advanced/combinatorial-testing.md)

<br/>

## ![✔] 7.4 Performance Testing

**TL;DR:** Although this is a vast topic, Performance testing from a web development perspective can be simplified with modern tools and understanding. It is highly effective in ensuring user experience, satisfying non-functional requirements (NFRS), and detecting possible system-flake early on.

🔗 [**Read More: Performance Testing**](./sections/advanced/performance-testing.md)

<br/>

## ![✔] 7.5 Email Testing

**TL;DR:** Email testing is [critical for business success](https://www.industrialmarketer.com/why-email-testing-is-critical-for-email-marketing-success/). Modern services not only allow automated email testing but also provide a stateless, scalable solution while testing SaaS applications.

🔗 [**Read More: Email Testing**](./sections/advanced/email-testing.md)

<br/><br/>

# `8. Real Life Examples`

## ![✔] 8.1 Siemens - Test the front-end with the integration tests, the back-end with the E2E ones - in reference to [Component vs Integration vs E2e Testing](./sections/testing-strategy/component-vs-integration-vs-e2e-testing.md)

**TL;DR:** UI tests with a stubbed server are reliable and faster compared to full E2E tests. Full E2E tests are not always necessary to ensure front-end quality. We can instead have high confidence in front-end quality by using lower-cost UI integration tests and saving higher cost E2E tests for the back-end.

**Otherwise:** You waste time and resources with slow and brittle E2E tests while you can get a lot of confidence with a lot of UI integrations tests.

🔗 [**Read More: Test the front-end with the integration tests, the back-end with the E2E ones**](./sections/real-life-examples/test-front-end-with-integration-back-end-with-e2e.md)

<br/>

## ![✔] 8.2 WorkWave - From unreadable React Component Tests to simple, stupid ones

**TL;DR:** The test's code must be as straightforward as possible. The benefit is to save a lot of time to understand, update, refactor, fix it when needed. At the opposite, a terrible scenario happens when you are not able to read some tests, even if you are the author! Here are reported some examples explaining why the test's code is hard, and how they have been refactored.

**Otherwise:** You waste a lot of time reading and understanding the tests when you have to update or fix them.

🔗 [**Read More: From unreadable React Component Tests to simple, stupid ones**](./sections/real-life-examples/from-unreadable-react-component-tests-to-simple-ones.md)

<br/> <br/>

# `9. Obsolete chapters`

## Unit Testing React components with Cypress

*This section is now marked as obsolete because it refers to a very old version of Cypress (that now fully supports component tests).*

**TL;DR:** Cypress v4.5.0 release allowed Unit Testing React components, an external tool like Storybook is not necessary anymore to test isolated components.

🔗 [**Read More: Unit Testing React components with Cypress.**](/sections/tools/cypress-react-component-test.md)

<br/>

## [@daedalius](https://github.com/daedalius)'s approach: Exposing components from Storybook separating stories from tests

*This section is now marked as obsolete because it refers to a very old version of Cypress and Storybook (either of them now fully support component tests).*

**TL;DR:** You may expose the component reference from Storybook Story to test it whatever you wish in Cypress without breaking testing logic into pieces.

**Otherwise:** Splitted test logic and test data will make it difficult to read and support.

🔗 [**Read More: Cypress + Storybook. Keeping test scenario, data and component rendering in one place.**](/sections/tools/cypress-and-storybook-exposing-component-from-story.md)

## [@NoriSte](https://github.com/NoriSte)'s approach: Testing a component with Cypress and Storybook

*This section is now marked as obsolete because it refers to a very old version of Cypress and Storybook (either of them now fully support component tests).*

**TL;DR:** Components ar the building blocks of your app, testing them in isolation is important to discover, as soon as possible, iof there is something wrong with them.

**Otherwise:** UI Tests without lower-level tests do not allow you to understand the source of the problem.

🔗 [**Read More: Testing a component with Cypress and Storybook**](/sections/tools/cypress-and-storybook.md)

<br/> <br/>

## Steering Committee

Meet the steering committee members - the people who work together to provide guidance and future direction to the project.

<div>
<img align="left" width="100" height="100" src="assets/images/members/noriste.png">

[Stefano Magni](https://github.com/NoriSte)
<a href="https://twitter.com/NoriSte"><img src="assets/images/twitter-s.png" width="16" height="16"></img></a>
<a href="https://github.com/NoriSte"><picture><source media="(prefers-color-scheme: dark)" srcset="assets/images/github-dark.png"><img alt="GitHub" src="assets/images/github-light.png" width="16" height="16"></picture>
<a href="https://www.linkedin.com/in/noriste/"><img src="assets/images/linkedin.png" width="16" height="16"></img></a>

</img></a>

Passionate, positive-minded / Front-end Tech Leader (platform) [Hasura](https://hasura.io/) / Speaker / Instructor / Remote worker.

<br/>
<br/>
</div>

<div>

<img align="left" width="100" height="100" src="assets/images/members/muratkeremozcan.png">

[Murat Ozcan](https://github.com/NoriSte)
<a href="https://twitter.com/MuratKeremOzcan"><img src="assets/images/twitter-s.png" width="16" height="16"></img></a>
<a href="https://github.com/muratkeremozcan"><picture><source media="(prefers-color-scheme: dark)" srcset="assets/images/github-dark.png"><img alt="GitHub" src="assets/images/github-light.png" width="16" height="16"></picture>
<a href="https://www.linkedin.com/in/murat-ozcan-3489898/"><img src="assets/images/linkedin.png" width="16" height="16"></img></a>

</img></a>

Tech enthusiast in love with testing, development, devops, web and cloud. Staff eng / Test architect at [Extend](https://www.extend.com/).

<br/>
<br/>

</div>

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
