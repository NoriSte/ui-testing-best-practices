<!-- markdownlint-disable MD041 -->
<!-- markdownlint-disable MD033 -->
<div align="right"><strong><a href="./README.md">🇬🇧 English version</a></strong>  | <strong>🇨🇳 中文 (Chinese version)</strong></div>
<!-- markdownlint-disable MD041 -->
<!-- markdownlint-disable MD033 -->

# UI 测试最佳实践

<h1 align="center">
  <img src="assets/images/banner-2.png" alt="UI testing Best Practices">
</h1>

<br/>

<div align="center">
  <img src="https://img.shields.io/badge/⚙%20Item%20count%20-%2026%20Chapters-blue.svg" alt="26 items"> <img src="https://img.shields.io/badge/%F0%9F%93%85%20Last%20update%20-%20Jun%202025-green.svg" alt="Last update: June, 2025">
</div>

<br/>

**在 Twitter 上关注我们！**:

- [**@NoriSte**](https://twitter.com/NoriSte/)
- [**@MuratKeremOzcan**](https://twitter.com/MuratKeremOzcan/)

<br/>

###### 由我们的[指导委员会](#指导委员会)和[合作者](#collaborators)构建和维护

## 目录

1. [测试策略 (5)](#1-测试策略)
2. [通用最佳实践 (6)](#2-通用最佳实践)
3. [服务通信测试 (3)](#3-服务通信测试)
4. [初学者 (1)](#4-初学者)
5. [通用测试的好处 (1)](#5-通用测试的好处)
6. [工具 (2)](#6-工具)
7. [进阶 (5)](#7-进阶)
8. [真实案例 (2)](#8-真实案例)
9. [过时章节 (3)](#9-过时章节)

<br/><br/>

# `1-测试策略`

## 1.1 组件测试 vs 集成测试 vs E2E 测试

**简而言之：** 辨识测试类型是理解和掌握所有 UI 测试策略、工具以及它们的利弊的起点。UI 集成测试是最有效的（你会喜欢上它们的），E2E 测试提供最高的信心，而组件测试则允许你独立测试 UI 的各个单元。

**反之：** 否则，你可能会陷入过多编写 E2E 测试的困境，而忽略其他更简单的测试类型。E2E 测试是最为可靠的测试类型，但同时也是最难、最慢且最脆弱的一种。

🔗 [**阅读更多：组件测试 vs (UI) 集成测试 vs E2E 测试**](/sections/testing-strategy/component-vs-integration-vs-e2e-testing.zh.md)

<br/>

## 1.2 在开始阶段，避免追求完美主义

**简而言之：** 软件测试是一个令人惊叹的话题，但有限的经验可能使你陷入与新敌人的斗争，而不是依赖新盟友。如果可能的话，在 UI 测试旅程的初期避免测试每个复杂的用户流程。你的第一个测试越简单，你越早获得优势。

**反之：** 你将创建复杂且难以调试的测试。这种类型的测试会拖慢你的工作，而且毫无用处。

🔗 [**阅读更多：在开始阶段，避免追求完美主义**](/sections/testing-strategy/avoid-perfectionism.zh.md)

<br/>

## 1.3 选择一个参考浏览器

**简而言之：** 跨浏览器测试被高估了。虽然这是一个重要的主题，也是在开始评估合适的测试工具时首先考虑的事项，但不必过于担心。首先，要将功能测试与视觉测试分开，这是正确评估是否需要跨浏览器支持的第一步，也是选择合适的测试工具的关键。视觉测试可以集成到每个测试工具中，这得益于诸如 Applitools 和 Percy 这样的服务。

**反之：** 基于跨浏览器支持做出选择，可能导致选择错误的测试工具。

🔗 [**阅读更多：选择一个参考浏览器**](/sections/testing-strategy/choose-a-reference-browser.zh.md)

<br/>

## 1.4 发现了 bug？先编写测试，然后再着手修复

**简而言之：** 在你需要确保能够有系统性地重现某个程序漏洞时，测试是一个极佳的助手。测试可以加速修复流程，同时让你百分之百确信同样的漏洞永远都能被捕捉到。

**反之：** 如果你不能正确地辨别漏洞，那么你无法确定这个漏洞将来是否还会再次出现。

🔗 [**阅读更多：发现了 bug？先编写测试，然后再着手修复**](/sections/testing-strategy/write-test-then-fix-bug.zh.md)

<br/>

## 1.5 单个长的端到端测试还是多个小的独立测试？

**简而言之：** 在处理端到端测试及其困难时，选择进行一次长时间的测试还是选择许多小而独立的测试并非显而易见。这两种解决方案都有各自的优劣，这源于端到端测试的内在复杂性，其中涉及真实后端和真实数据。

**反之：** 你可能会创建难以维护的端到端测试。

🔗 [**阅读更多：单个长的端到端测试还是多个小的独立测试？**](/sections/testing-strategy/small-tests-or-long-ones.zh.md)

<br/><br/>

# `2-通用最佳实践`

## 2.1 等待，不要休眠

**简而言之：** 在测试用户界面时，你要定义应用程序必须经过的关键点。达到这些关键点是一个异步的过程，因为几乎百分之百的情况下，用户界面不会同步更新。这些关键点被称为**确定性事件**，即你知道必须发生的事情。你需要等待这些事件以确保你的测试更加健壮。

**反之：** 让测试休眠会使测试变得缓慢而脆弱，这是用户界面测试中最常见且最严重的错误之一。

🔗 [**阅读更多：等待，不要休眠**](/sections/generic-best-practices/await-dont-sleep.zh.md)

<br/>

## 2.2 明智地为测试文件命名

**简而言之：** 很多时候，你可能只需要运行某一类测试，如果你在为测试文件命名时遵循一种常见的模式，那将非常方便。

**反之：** 你可能需要运行一个冗长的测试套件，而实际上只是为了运行其中的一些测试。

🔗 [**阅读更多：明智地为测试文件命名**](/sections/generic-best-practices/name-test-files-wisely.zh.md)

<br/>

## 2.3 UI 测试调试最佳实践

**简而言之：** 调试 UI 测试可能非常困难，特别是当你使用通用的浏览器自动化工具时。以下是调试过程中的一些基本规则。

**反之：** 你将会花费大量时间，而无法应对 UI 测试的指数复杂性。

🔗 [**阅读更多：UI 测试调试最佳实践**](/sections/generic-best-practices/ui-tests-debugging-best-practices.zh.md)

<br/>

## 2.4 在测试中达到 UI 状态而无需使用 UI

**简而言之：** 作为一个追求质量的开发者，思考测试的成本与它们提供的价值是至关重要的。在合理的情况下，努力避免重复努力，并通过考虑为测试设置状态的替代方案，依然能够获得高价值。

🔗 [**阅读更多：达到 UI 状态**](./sections/generic-best-practices/reaching-ui-state.zh.md)

<br/>

## 2.5 将你的测试工具用作主要的开发工具

**简而言之：** 利用测试工具来避免手动测试是提高工作效率的最大改进之一。测试工具比你更快，而且大多数现代工具都包含一些 UI 工具，使得将其用作开发工具变得更加容易。

**反之：** 以传统方式编写应用程序，花费大量时间手动与 UI 进行交互。

🔗 [**阅读更多：将你的测试工具用作主要的开发工具**](/sections/generic-best-practices/use-your-testing-tool-as-your-primary-development-tool.zh.md)

<br/>

## 2.6 保持低抽象度以便于调试测试

**简而言之：** 编写测试时应考虑可读性和可调试性。在某些情况下，抽象可能是有益的，但它总是会增加调试的成本，因此有时可能不值得。这对于 UI 测试尤为重要；由于复杂的技术栈，理解故障的真实源头可能变得更加困难。为了更容易调试，降低抽象度是未来测试代码的关键。

**反之：** 抽象度和可调试性之间存在平衡；抽象度越高，将来调试测试就越困难。

🔗 [**阅读更多：保持低抽象度以便于调试测试**](/sections/generic-best-practices/test-code-with-debugging-in-mind.zh.md)

<br/><br/>

# `3-服务通信测试`

## 3.1 检验请求和响应负载

**简而言之：** UI 与后端持续通信，通常每次通信都至关重要。不良的请求或响应可能导致不一致的数据和不一致的 UI 状态。请记住，所有业务都围绕数据构建，而每次 UI 失败都会影响用户体验。因此，必须仔细检查每个 XHR 请求。XHR 请求的检查还能使测试更为健壮，正确的 XHR 管理和测试是 UI 测试中最重要的方面之一。

**反之：** 你可能会错过一些相关的通信不一致性，当你需要调试时，由于测试不会直接指引你找到问题，你将浪费大量时间。

🔗 [**阅读更多：检验请求和响应负载**](/sections/server-communication-testing/test-request-and-response-payload.zh.md)

<br/>

## 3.2 审查服务器架构

**简而言之：** 前端应用很多时候会因后端的变化而出现故障。请向后端同事请求允许你导出描述后端实体和与前端通信的每个架构的所有模式。一些示例可能包括 GraphQL 架构、TypeScript 类型、ElasticSearch 映射、Pact 合同、Postman 配置等，更一般地说，一切都可以提醒你后端发生了变化。每次后端更改都可能影响前端，你必须尽早发现。

**反之：** 你可能会错过一些后端更改，导致前端应用不经意间崩溃。

<br/>

## 3.3 测试监控

**简而言之：** 测试套件定期启动的次数越多，你对一切都按预期工作的信心就越足。UI 测试应该基于用户的视角，但有许多小测试可以为你提供大量即时反馈，而无需调试用户流程。监控那些看似微不足道的技术细节有助于预防更大的测试失败。

**反之：** 你会将技术细节测试与用户导向的测试混为一谈。

🔗 [**阅读更多：测试监控**](/sections/server-communication-testing/monitoring-tests.zh.md)

<br/><br/>

# `4-初学者`

## 4.1 从金字塔顶层入手测试

**简而言之：** 以金字塔顶层作为测试入手可能效率低下且不令人满意。你开始编写一些单元测试，但仍然存在许多疑虑。通过 UI 测试，你可以从第一天就以较高的信心开始测试。

**反之：** 错误的方法可能会影响你对测试的思考方式，使你产生错误的测试方式的错误观念，而实际上你并没有进行有效的测试。

🔗 [**阅读更多：从金字塔顶层入手测试！**](/sections/beginners/top-to-bottom-approach.zh.md)

<br/><br/>

# `5-通用测试的好处`

## 5.1 将测试视为文档工具

**简而言之：** 测试是一种简洁、与代码紧密关联且不断更新的文档方式。通过具有良好叙述的测试描述，可以使对代码库或新项目的理解变得非常简单。

**反之：** 如果不依赖于代码文档，甚至更糟糕的是依赖于代码的可读性来理解代码的作用。

🔗 [**阅读更多：将测试视为文档工具**](/sections/testing-perks/tests-as-documentation.zh.md)

<br/><br/>

# `6-工具`

## 6.1 一些 UI 测试问题及 Cypress 的解决方案

**简而言之：** 为什么测试 Web 应用这么难？通用的浏览器自动化工具为什么不太适用于 UI/E2E 测试的需求？Cypress 为什么如此出色？

**否则：** 仅仅通过通用功能比较无法理解 UI 测试的主要问题，以及 Cypress 是如何解决它们的。

> [!NOTE]
> 所有现代前端测试工具（Playwright、Storybook、Cypress、TestCafé）都具有类似的功能和 UI 工具，因此相同的内容适用于所有工具，而不仅仅是 Cypress。

🔗 [**阅读更多：一些 UI 测试问题及 Cypress 的解决方案**](/sections/tools/ui-testing-problems-cypress.zh.md)

<br/><br/>

## 6.2 视觉回归测试

**简而言之：** 视觉回归测试为什么如此困难，以及为什么我们应该依赖高级服务。

**反之：** 又是一个我们不在意的回归测试工作。有可能会漏掉视觉差异。

🔗 [**阅读更多：视觉回归测试**](/sections/tools/visual-regression-testing.zh.md)

<br/><br/>

# `7-进阶`

## 7.1 测试状态

**简而言之：** 测试应该是可重复的、模块化的，并且应该处理自身的状态设置。为了实现其他测试的状态，不应重复执行 UI 测试。

🔗 [**阅读更多：测试状态**](./sections/advanced/test-states.zh.md)

<br/>

## 7.2 不稳定的测试

**简而言之：** 每次测试应产生一致的结果。可重复的流水线执行结果是选举的基准。
如果测试无法产生可靠的结果，它会降低对测试的信心并需要维护，从而降低所有价值。在这些情况下，最好手动测试功能。

🔗 [**阅读更多：不稳定的测试**](./sections/advanced/test-flake.zh.md)

<br/>

## 7.3 组合测试

**简而言之：** 大多数软件错误和故障是由一个或两个参数引起的。测试参数组合可以比传统方法更有效地检测故障。组合测试是一种经过验证的、成本更低的更有效的软件测试方法。

🔗 [**阅读更多：组合测试**](./sections/advanced/combinatorial-testing.zh.md)

<br/>

## 7.4 性能测试

**简而言之：** 尽管这是一个庞大的主题，但从 Web 开发的角度来看，性能测试可以通过现代工具和理解来简化。它在确保用户体验、满足非功能性需求（NFRS）以及及早检测可能的系统故障方面非常有效。

🔗 [**阅读更多：性能测试**](./sections/advanced/performance-testing.zh.md)

<br/>

## 7.5 电子邮件测试

**简而言之：** 电子邮件测试对于业务成功至关重要。现代服务不仅允许自动化的电子邮件测试，而且在测试 SaaS 应用程序时还提供了一种无状态、可扩展的解决方案。

🔗 [**阅读更多：电子邮件测试**](./sections/advanced/email-testing.zh.md)

---

<br/><br/>

# `8-真实案例`

## 8.1 Siemens - 用集成测试测试前端，用 E2E 测试测试后端 - 参考 [组件测试 vs（UI）集成测试 vs E2E 测试](./sections/testing-strategy/component-vs-integration-vs-e2e-testing.zh.md)

**简而言之：** 使用带有存根服务器的 UI 测试相比完整的 E2E 测试更为可靠且更快。并非总是需要完整的 E2E 测试来确保前端质量。我们可以通过使用成本较低的 UI 集成测试对前端质量具有高度信心，并将成本较高的 E2E 测试保留给后端。

**反之：** 在进行缓慢且脆弱的 E2E 测试时浪费时间和资源，而通过进行大量的 UI 集成测试可以获得高度的信心。

🔗 [**阅读更多：用集成测试测试前端，用 E2E 测试测试后端**](./sections/real-life-examples/test-front-end-with-integration-back-end-with-e2e.zh.md)

<br/>

## 8.2 WorkWave - 从难以理解的 React 组件测试到简单愚蠢的测试

**简而言之：** 测试代码必须尽可能简单明了。这样可以在需要时节省大量的理解、更新、重构和修复时间。相反，如果甚至作者自己也无法阅读某些测试，那就会发生可怕的情况！这里提供了一些解释测试代码难以理解的例子，并展示了它们的重构过程。

**反之：** 当您必须更新或修复测试时，花费大量时间阅读和理解测试。

🔗 [**阅读更多：从难以理解的 React 组件测试到简单愚蠢的测试**](./sections/real-life-examples/from-unreadable-react-component-tests-to-simple-ones.zh.md)

<br/> <br/>

# `9-过时章节`

## 使用 Cypress 进行 React 组件的单元测试

*此部分现已标记为过时，因为它涉及到一个非常旧的 Cypress 版本（现在已完全支持组件测试）。

**简而言之：** Cypress v4.5.0 发布允许对 React 组件进行单元测试，不再需要类似 Storybook 的外部工具来测试孤立的组件。

🔗 [**阅读更多：使用 Cypress 进行 React 组件的单元测试**](/sections/tools/cypress-react-component-test.zh.md)

<br/>

## [@daedalius](https://github.com/daedalius) 的方法：从 Storybook 中公开组件，将故事与测试分离

*此部分现已标记为过时，因为它涉及到一个非常旧的 Cypress 和 Storybook 版本（现在它们都完全支持组件测试）。

**简而言之：** 你可以从 Storybook 故事中公开组件引用，以在 Cypress 中测试任何你希望测试的内容，而不会将测试逻辑分解成多个部分。

**反之：** 将测试逻辑和测试数据拆分开会使其难以阅读和维护。

🔗 [**阅读更多：Cypress + Storybook。保持测试场景、数据和组件渲染在一个地方。**](/sections/tools/cypress-and-storybook-exposing-component-from-story.zh.md)

## [@NoriSte](https://github.com/NoriSte) 的方法：使用 Cypress 和 Storybook 进行组件测试

*此部分现已标记为过时，因为它涉及到一个非常旧的 Cypress 和 Storybook 版本（现在它们都完全支持组件测试）。

**简而言之：** 组件是你的应用程序的构建块，在孤立状态下测试它们对于尽早发现问题非常重要。

**反之：** 没有底层测试的 UI 测试无法让你了解问题的根源。

🔗 [**阅读更多：使用 Cypress 和 Storybook 进行组件测试**](/sections/tools/cypress-and-storybook.zh.md)

<br/> <br/>

## 指导委员会

了解指导委员会成员 - 一起为项目提供指导和未来方向的人们。

<div>
<img align="left" width="100" height="100" src="assets/images/members/noriste.png">

[Stefano Magni](https://github.com/NoriSte)
<a href="https://twitter.com/NoriSte"><img src="assets/images/twitter-s.png" width="16" height="16"></img></a>
<a href="https://github.com/NoriSte"><picture><source media="(prefers-color-scheme: dark)" srcset="assets/images/github-dark.png"><img alt="GitHub" src="assets/images/github-light.png" width="16" height="16"></picture>
<a href="https://www.linkedin.com/in/noriste/"><img src="assets/images/linkedin.png" width="16" height="16"></img></a>

</img></a>

充满激情，积极乐观 / 前端技术领导者（平台）[Hasura](https://hasura.io/) / 演讲者 / 讲师 / 远程工作者。

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

热衷于技术，热爱测试、开发、DevOps、Web 和云。[Extend](https://www.extend.com/) 的技术专家 / 测试架构师。

<br/>
<br/>

</div>

## 感谢信

我们感激任何贡献，无论是单个词汇的修复还是新的最佳实践。下面是为这个项目做出贡献的所有人的列表。🌻标记着成功的拉取请求，⭐标记着被批准的新最佳实践。

### Stars

一个被批准的新最佳实践，成为第一个收到⭐的人，为这个仓库做出贡献 😁

⭐ [Murat Ozcan](https://github.com/muratkeremozcan)
⭐ [Dmitriy Tishin](https://github.com/daedalius)

### 鲜花

一个成功的 PR 会给你一朵 🌻，成为第一个收到它的人。

🌻 [Anoop Kumar Gupta](https://github.com/anoop-gupt)
🌻 [Ferdinando Santacroce](https://github.com/jesuswasrasta)
🌻 [Luca Piazzoni](https://github.com/bioz87)
🌻 [Luca Previtali](https://www.linkedin.com/in/previtaliluca/)
🌻 [Luca Previtali](https://www.linkedin.com/in/previtaliluca/)
🌻 [Filip Hric](https://github.com/filiphric)

<br/><br/><br/>

这个仓库受到了 [nodebestpractices](https://github.com/i0natan/nodebestpractices) 仓库的启发，感谢 [Yoni](https://github.com/i0natan) 和整个 [指导委员会](https://github.com/i0natan/nodebestpractices#steering-committee) 的努力，保持其更新并允许创建这个仓库。

<br/><br/><br/>
