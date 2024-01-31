# 明智地为测试文件命名

<br/><br/>

## 一段简要说明

编写各种不同的 UI 测试是一种好习惯，而采用一种常见的测试文件命名方式更是有益的。

这很有用，因为通常情况下，你需要仅运行某一类测试，可能的情况包括：

- 在开发过程中，你只需要运行其中一些测试
  - 你正在更改一些相关组件，并需要检查生成的标记是否发生了变化
  - 你正在更改全局 CSS 规则，只需运行视觉测试
  - 你正在更改应用程序流程，需要运行整个应用程序集成测试
- 你的 DevOps 同事需要确保一切正常运行，最简单的方法就是只运行端对端测试
- 你的构建流水线需要运行集成测试和端对端测试
- 你的监控流水线需要一个脚本来运行端对端测试和监控测试

如果你为测试取一个明智的命名，将会非常容易只运行其中的某些类型。

Cypress:

```bash
cypress run --spec \"cypress/integration/**/*.e2e.*\"
```

Jest:

```bash
jest --testPathPattern=e2e\\.*$
```

<br>

没有一种全局的命名测试文件的方式，一个建议是使用以下方式命名：

- 正在测试的主题
- 测试的类型（`integration`、`e2e`、`monitoring`、`component`等）
- 选择的测试后缀（`test`、`spec`等）
- 文件扩展名（`.js`、`.ts`、`.jsx`、`.tsx`等）

它们之间用句点分隔。

以下是一些例子：

- `authentication.e2e.test.ts`
- `authentication.integration.test.ts`
- `site.monitoring.test.js`
- `login.component.test.tsx`
等等。
