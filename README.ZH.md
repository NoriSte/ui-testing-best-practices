<!-- markdownlint-disable MD041 -->
<!-- markdownlint-disable MD033 -->
<div align="right"><strong>🇨🇳中文</a></strong>  | <strong><a href="./README.md">🇬🇧English</strong></div>
<!-- markdownlint-disable MD041 -->
<!-- markdownlint-disable MD033 -->

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

1. [测试策略 (5)](#1-测试策略)
2. [通用最佳实践 (6)](#2-通用最佳实践)
3. [服务通信测试 (3)](#3-服务通信测试)
4. [初学者 (1)](#4-初学者)
5. [通用测试的优势 (1)](#5-generic-testing-perks)
6. [工具 (2)](#6-tools)
7. [高级 (5)](#7-advanced)
8. [真实案例 (2)](#8-real-life-examples)
9. [过时章节 (3)](#9-obsolete-chapters)

<br/><br/>

# 1-测试策略

## ![✔] 1.1 组件测试 vs 集成测试 vs E2E 测试

**简而言之：** 辨识测试类型是理解和掌握所有 UI 测试策略、工具以及它们的利弊的起点。UI 集成测试是最有效的（你会喜欢上它们的），E2E 测试提供最高的信心，而组件测试则允许你独立测试 UI 的各个单元。

**否则：** 否则，你可能会陷入过多编写 E2E 测试的困境，而忽略其他更简单的测试类型。E2E 测试是最为可靠的测试类型，但同时也是最难、最慢且最脆弱的一种。

🔗 [**阅读更多：组件测试 vs (UI) 集成测试 vs E2E 测试**](/sections/testing-strategy/component-vs-integration-vs-e2e-testing.zh.md)

<br/>

## ![✔] 1.2 在开始阶段，避免追求完美主义

**简而言之：** 软件测试是一个令人惊叹的话题，但有限的经验可能使您陷入与新敌人的斗争，而不是依赖新盟友。如果可能的话，在 UI 测试旅程的初期避免测试每个复杂的用户流程。您的第一个测试越简单，您越早获得优势。

**否则：** 您将创建复杂且难以调试的测试。这种类型的测试会拖慢您的工作，而且毫无用处。

🔗 [**阅读更多：在开始阶段，避免追求完美主义**](/sections/testing-strategy/avoid-perfectionism.zh.md)

<br/>

## ![✔] 1.3 选择一个参考浏览器

**简而言之：** 跨浏览器测试被高估了。虽然这是一个重要的主题，也是在开始评估合适的测试工具时首先考虑的事项，但不必过于担心。首先，要将功能测试与视觉测试分开，这是正确评估是否需要跨浏览器支持的第一步，也是选择合适的测试工具的关键。视觉测试可以集成到每个测试工具中，这得益于诸如 Applitools 和 Percy 这样的服务。

**否则：** 基于跨浏览器支持做出选择，可能导致选择错误的测试工具。

🔗 [**阅读更多：选择一个参考浏览器**](/sections/testing-strategy/choose-a-reference-browser.zh.md)

<br/>

## ![✔] 1.4 发现了 bug？先编写测试，然后再着手修复

**简而言之：** 在你需要确保能够有系统性地重现某个程序漏洞时，测试是一个极佳的助手。测试可以加速修复流程，同时让你百分之百确信同样的漏洞永远都能被捕捉到。

**否则：** 如果你不能正确地辨别漏洞，那么你无法确定这个漏洞将来是否还会再次出现。

🔗 [**阅读更多：发现了 bug？先编写测试，然后再着手修复**](/sections/testing-strategy/write-test-then-fix-bug.zh.md)

<br/>

## ![✔] 1.5 单个长的端到端测试还是多个小的独立测试？

**简而言之：** 在处理端到端测试及其困难时，选择进行一次长时间的测试还是选择许多小而独立的测试并非显而易见。这两种解决方案都有各自的优劣，这源于端到端测试的内在复杂性，其中涉及真实后端和真实数据。

**否则：** 你可能会创建难以维护的端到端测试。

🔗 [**阅读更多：单个长的端到端测试还是多个小的独立测试？**](/sections/testing-strategy/small-tests-or-long-ones.zh.md)

<br/><br/>

# 2-通用最佳实践

## ![✔] 2.1 等待，不要休眠

**简而言之：** 在测试用户界面时，您要定义应用程序必须经过的关键点。达到这些关键点是一个异步的过程，因为几乎百分之百的情况下，用户界面不会同步更新。这些关键点被称为**确定性事件**，即您知道必须发生的事情。您需要等待这些事件以确保您的测试更加健壮。

**否则：** 让测试休眠会使测试变得缓慢而脆弱，这是用户界面测试中最常见且最严重的错误之一。

🔗 [**阅读更多：等待，不要休眠**](/sections/generic-best-practices/await-dont-sleep.zh.md)

<br/>

## ![✔] 2.2 明智地为测试文件命名

**简而言之：** 很多时候，您可能只需要运行某一类测试，如果您在为测试文件命名时遵循一种常见的模式，那将非常方便。

**否则：** 您可能需要运行一个冗长的测试套件，而实际上只是为了运行其中的一些测试。

🔗 [**阅读更多：明智地为测试文件命名**](/sections/generic-best-practices/name-test-files-wisely.zh.md)

<br/>

## ![✔] 2.3 UI 测试调试最佳实践

**简而言之：** 调试 UI 测试可能非常困难，特别是当您使用通用的浏览器自动化工具时。以下是调试过程中的一些基本规则。

**否则：** 您将会花费大量时间，而无法应对 UI 测试的指数复杂性。

🔗 [**阅读更多：UI 测试调试最佳实践**](/sections/generic-best-practices/ui-tests-debugging-best-practices.zh.md)

<br/>

## ![✔] 2.4 在测试中达到 UI 状态而无需使用 UI

**简而言之：** 作为一个追求质量的开发者，思考测试的成本与它们提供的价值是至关重要的。在合理的情况下，努力避免重复努力，并通过考虑为测试设置状态的替代方案，依然能够获得高价值。

🔗 [**阅读更多：达到 UI 状态**](./sections/generic-best-practices/reaching-ui-state.zh.md)

<br/>

## ![✔] 2.5 将您的测试工具用作主要的开发工具

**简而言之：** 利用测试工具来避免手动测试是提高工作效率的最大改进之一。测试工具比你更快，而且大多数现代工具都包含一些 UI 工具，使得将其用作开发工具变得更加容易。

**否则：** 以传统方式编写应用程序，花费大量时间手动与 UI 进行交互。

🔗 [**阅读更多：将您的测试工具用作主要的开发工具**](/sections/generic-best-practices/use-your-testing-tool-as-your-primary-development-tool.zh.md)

<br/>

## ![✔] 2.6 保持低抽象度以便于调试测试

**简而言之：** 编写测试时应考虑可读性和可调试性。在某些情况下，抽象可能是有益的，但它总是会增加调试的成本，因此有时可能不值得。这对于 UI 测试尤为重要；由于复杂的技术栈，理解故障的真实源头可能变得更加困难。为了更容易调试，降低抽象度是未来测试代码的关键。

**否则：** 抽象度和可调试性之间存在平衡；抽象度越高，将来调试测试就越困难。

🔗 [**阅读更多：保持低抽象度以便于调试测试**](/sections/generic-best-practices/test-code-with-debugging-in-mind.zh.md)

<br/><br/>

# 3-服务通信测试

## ![✔] 3.1 检验请求和响应负载

**简而言之：** UI 与后端持续通信，通常每次通信都至关重要。不良的请求或响应可能导致不一致的数据和不一致的 UI 状态。请记住，所有业务都围绕数据构建，而每次 UI 失败都会影响用户体验。因此，必须仔细检查每个 XHR 请求。XHR 请求的检查还能使测试更为健壮，正确的 XHR 管理和测试是 UI 测试中最重要的方面之一。

**否则：** 您可能会错过一些相关的通信不一致性，当您需要调试时，由于测试不会直接指引您找到问题，您将浪费大量时间。

🔗 [**阅读更多：检验请求和响应负载**](/sections/server-communication-testing/test-request-and-response-payload.zh.md)

<br/>

## ![✔] 3.2 审查服务器架构

**简而言之：** 前端应用很多时候会因后端的变化而出现故障。请向后端同事请求允许您导出描述后端实体和与前端通信的每个架构的所有模式。一些示例可能包括 GraphQL 架构、TypeScript 类型、ElasticSearch 映射、Pact 合同、Postman 配置等，更一般地说，一切都可以提醒您后端发生了变化。每次后端更改都可能影响前端，您必须尽早发现。

**否则：** 您可能会错过一些后端更改，导致前端应用不经意间崩溃。

<br/>

## ![✔] 3.3 测试监控

**简而言之：** 测试套件定期启动的次数越多，您对一切都按预期工作的信心就越足。UI 测试应该基于用户的视角，但有许多小测试可以为您提供大量即时反馈，而无需调试用户流程。监控那些看似微不足道的技术细节有助于预防更大的测试失败。

**否则：** 您会将技术细节测试与用户导向的测试混为一谈。

🔗 [**阅读更多：测试监控**](/sections/server-communication-testing/monitoring-tests.zh.md)

<br/><br/>

# 4-初学者

## ![✔] 4.1 从金字塔顶层入手测试

**简而言之：** 以金字塔顶层作为测试入手可能效率低下且不令人满意。你开始编写一些单元测试，但仍然存在许多疑虑。通过 UI 测试，你可以从第一天就以较高的信心开始测试。

**否则：** 错误的方法可能会影响你对测试的思考方式，使你产生错误的测试方式的错误观念，而实际上你并没有进行有效的测试。

🔗 [**阅读更多：从金字塔顶层入手测试！**](/sections/beginners/top-to-bottom-approach.zh.md)

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
