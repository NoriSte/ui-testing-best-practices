# 选择一个参考浏览器

<br/>

## 一段简要说明

每个人都关心跨浏览器测试。我们通常习惯在每个浏览器上手动测试所有内容，因为我们知道，不同浏览器之间存在许多差异。当我们开始评估合适的测试工具时，跨浏览器测试是一个重要的话题，也是你在考虑时可能首先想到的。但是不要担心：首先从功能测试和视觉测试分离开始，这是正确评估跨浏览器支持需求（也是选择正确测试工具的第一步）。视觉测试可以集成到每个测试工具中，感谢诸如 Applitools 和 Percy 这样的服务。

换句话说，不要仅仅基于跨浏览器支持来选择测试工具。以下是一些建议：

- **Selenium 和 Puppeteer 是通用的自动化工具**。它们可以用作测试工具（有许多插件和模块可帮助你实现），但它们并非专为测试而设计，因此它们缺少一些集成实用工具，这可能使测试编写更加简便。

- 只考虑 **Cypress、Playwright 和 TestCafé**，因为它们是专为**简化 UI 测试过程**而创建的工具。这些工具自动处理一半的最佳实践，而在测试中的一些方面，它们可能更符合你的需求。在 UI 测试方面，由于其

困难性，花些时间试验这些工具是值得的。

- 仔细思考你需要测试什么。如果你需要测试特定的移动能力，请选择 [TestCafé](https://testcafe.devexpress.com)，但如果你只需要测试表单和按钮是否正常工作，你在选择上就更加灵活。

- 查看 [Cypress Test Runner](https://docs.cypress.io/guides/core-concepts/test-runner.html#Command-Log)，这是使 Cypress 异于常人的工具，对于测试开发过程中非常有帮助。

- 研究 [Playwright 在调试方面的优势](https://playwright.dev/docs/trace-viewer-intro)。Playwright 非常快速稳定，最近其开发体验有了很大改进。

- 跨浏览器测试通常涉及到视觉测试（CSS 浏览器差异），但这与功能测试不同。视觉测试得益于许多专用插件和工具的支持。详细了解 [视觉测试对应的章节](../tools/visual-regression-testing.zh.md) [Applitools](https://applitools.com)，其中我们讨论了一些专用产品，这些产品可以与几乎所有测试工具集成，通过将被测试页面的快照上传到其服务器并进行呈现来进行工作。

你还可以在 [等待，不是休眠](/sections/generic-best-practices/await-dont-sleep.zh.md) 章节中了解各种测试工具之间的一些差异。
