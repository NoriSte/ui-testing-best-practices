# 将您的测试工具用作主要的开发工具

<br/><br/>

## 一段简要说明

一个实例展示出问题的本质。比如说，你正在开发一个身份验证表单，可能的步骤是：

- 编写用户名输入字段的代码
- **在浏览器中手动测试**它
- 编写密码输入字段的代码
- **在浏览器中手动测试**它
- 编写提交按钮的代码
- 编写 XHR 请求的处理代码

然后，你遇到了一个问题，因为在不修改源代码的情况下，你需要一个虚拟的或模拟的服务器来响应应用程序的 XHR 请求。**于是，你开始编写一个集成测试**：

- 填写用户名输入字段
- 填写密码输入字段
- 点击提交按钮
- 检查 XHR 请求
- 模拟 XHR 的响应
- 检查反馈
- 对每个错误流程进行相同的测试步骤
- 编写处理错误流程的代码
- 重新检查测试结果

看一下第一个测试步骤，它们与我们在编写身份验证表单时**手动执行的步骤相同**。然后，我们为服务器的响应创建了存根，并检查表单的最终行为（包括成功或失败的响应）。

这个工作流程可以很容易地得到改进，如果我们在编写表单的同时编写测试（TDD 开发者已经训练成这样）：

- **编写用户名输入字段的代码** *
- **编写填充**用户名输入字段的测试*
- 编写密码输入字段的代码
- 更新测试以填充密码输入字段
- 编写提交按钮的代码
- 更新测试以点击提交按钮
- 在测试中创建 XHR 的响应存根
- 编写 XHR 请求的管理代码
- 检查反馈
- 编写错误流程的管理代码
- 更新测试以检查错误流程
- 对于每个错误流程，重复以上步骤

\* 请注意，如果要采用严格的 TDD 方法，可以颠倒第一步和第二步的顺序。

这样做的最重要的好处是什么？

- 你**几乎完全避免手动测试**应用程序
- 充分利用测试工具的速度，它以惊人的速度填充表单，让你**节省大量时间**
- 无需在编写表单后再编写测试（TDD 开发者已经避免这样做），尽管最初可能看起来有点烦人
- 完全**避免在源代码中引入一些临时状态**（例如输入字段的默认值、虚假的 XHR 响应）
- 直接用真实的网络响应测试你的应用程序（请记住，应用程序不知道网络请求是由测试工具存根的）
- 每次保存测试文件时都重新启动测试
- 可以充分利用 Chrome DevTools 和框架特定的开发工具

如何充分利用**现有的开发工具**？<br>
嗯，几乎每个测试工具都可以做到这一点，但 Cypress 在这方面脱颖而出。Cypress 拥有一个专门的 Chrome 用户，该用户在所有你的测试和所有你的项目中都是持久存在的。通过这样做，Cypress 允许你拥有一个真正专为测试而设的浏览器，其中包含你喜欢的扩展，即使你使用的是与浏览相同的 Chrome 版本。<br>
将其与出色的用户界面结合起来，你就可以准备好直接使用 Cypress 开发应用程序。下面你可以看到 Cypress 用户界面的一些截图，展示了将其作为主要开发工具使用的简便性。

<br>

**浏览器选择**
![Cypress 浏览器选择](../../assets/images/use-your-testing-tool-as-your-primary-development-tool/browser-selection.png
"Cypress 浏览器选择")

**Cypress 控制的浏览器开发者工具**
![Cypress 浏览器开发者工具](../../assets/images/use-your-testing-tool-as-your-primary-development-tool/devtools.jpg
"Cypress 浏览器开发者工具")

**Cypress [Skip 和 Only UI 插件](https://github.com/bahmutov/cypress-skip-and-only-ui)** 这个工具让你可以直接在 Cypress UI 中为测试添加`.only`或`.skip`。
![Cypress Skip 和 Only UI](../../assets/images/use-your-testing-tool-as-your-primary-development-tool/skip-and-only.gif
"Cypress Skip 和 Only UI")

**Cypress [观察和重新加载插件](https://github.com/bahmutov/cypress-watch-and-reload)** 此功能使您能够在每次源代码编译时重新运行 Cypress 测试。

如果您想在 Cypress 控制的浏览器中查看 React/Redux 开发工具的实际效果，可以使用 [cypress-react-devtools](https://github.com/NoriSte/cypress-react-devtools) 存储库。

## 相关章节

- 🔗 [发现了 bug？先编写测试，然后再着手修复](/sections/testing-strategy/write-test-then-fix-bug.zh.md)

<br /><br />

*此文由 [NoriSte](https://github.com/NoriSte) 在 [dev.to](https://dev.to/noriste/front-end-productivity-boost-cypress-as-your-main-development-browser-5cdk) 和 [Medium](https://medium.com/@NoriSte/front-end-productivity-boost-cypress-as-your-main-development-browser-f08721123498)上进行联合发表.*
