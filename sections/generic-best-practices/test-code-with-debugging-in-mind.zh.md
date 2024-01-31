# 保持低抽象度以便于调试测试

<br/><br/>

## 一段简要说明

UI 测试涉及许多步骤，主要有三个关键目标，但其中两个往往被低估：

1. **测试一个功能**（显而易见）
2. **帮助读者理解代码的作用**（通常被低估）
3. **简化调试**（被低估，同时需要经验）

下面让我们一起了解编写 UI 测试时要记住的一些简单但有效的技巧。

<br/><br/>

## 可读性

关于测试中抽象的问题是一个有争议的话题（Page-Object Model 的粉丝可能会对此有异议）。

让我们看一个我不得不修复的真实测试的例子。

```ts
// spec.ts file
it('Create Query Action', createQueryAction);

// test.ts file (simplified version)
export const createMutationAction = () => {
  // ...
  clearActionDef();
  typeIntoActionDef(statements.createMutationActionText);
  clearActionTypes();
  // ...
};

// test.ts file contains the clearActionDef, typeIntoActionDef, etc.
const clearActionDef = () => {
  cy.get('textarea').first().type('{selectall}', { force: true });
  cy.get('textarea').first().trigger('keydown', {
    keyCode: 46,
    which: 46,
    force: true,
  });
};

// test.ts file contains also the statements
const statements = {
  createMutationActionText: `type Mutation {
    login (username: String!, password: String!): LoginResponse
  }`,
  createMutationCustomType: `type LoginResponse {
    accessToken: String!
  }
  `,
  createMutationHandler: 'https://hasura-actions-demo.glitch.me/login',
  // ...
}
```

短函数的背后思路是创建小而可重复使用的代码片段，以帮助其他需要在页面上执行类似操作的测试。

我认为这不太好，因为**很难建立对测试的执行过程的心智模型**！所有测试部分都被分割成小函数和实用程序，而测试的代码必须尽可能地直截了当。

你还记得章节开头提到的两个被低估的点吗？这个想法是测试应该实现三个主要目标：

- 帮助读者理解代码的作用
- 以便轻松进行调试

前者要求测试的代码尽可能简单，而在测试的代码中使用抽象并没有好处，因为这会导致花费更多时间调试和维护测试，而不是应用程序。

后者与测试的错误部分有关：调试/修复它们。调试 UI 测试很困难，因为你需要处理以下元素：

- 你的前端应用程序
- 浏览器
- 控制浏览器的工具
- 你提供给控制浏览器的工具的指令

上述每个元素都可能出现问题，即使是经验丰富的开发人员也可能在理解测试失败的原因时感到困扰。

因此，端到端测试是复杂的。Cypress 提高了开发人员的生活质量（在 [一些 UI 测试问题和 Cypress 方法](../tools/ui-testing-problems-cypress.zh.md) 章节中了解更多），但直截了当的代码会极大地帮助。

## 不使用任何抽象

我建议根本不使用抽象（稍后，我将讨论一些例外情况以及哪种抽象是好的）！我将上述例子改写为如下形式：

```ts
it('Test the feature', () => {
  cy.get('textarea').eq(0).as('actionDefinitionTextarea');
  cy.get('textarea').eq(1).as('typeConfigurationTextarea');

  cy.get('@actionDefinitionTextarea').clearConsoleTextarea().type(
    `type Mutation {
        login (username: String!, password: String!): LoginResponse
      }`,
    { force: true, delay: 0 }
  );

  cy.get('@typeConfigurationTextarea').clearConsoleTextarea().type(
    `type LoginResponse {
      accessToken: String!
    }
    `,
    { force: true, delay: 0 }
  );

  // ...
})
```

重写后的测试与原始测试执行相同的操作，但当你查看测试代码时，无需来回跳转来在脑中建立连接。

- 想知道在文本区域中输入了什么吗？毫不费力，就在那里！
- 想知道文本使用的是哪个文本区域吗？毫不费力，就在那里！

## 在测试中什么时候使用抽象化是好的呢？

在我看来：

- 当我想隐藏一些可能没有价值但可能分散读者注意力的测试怪癖时
- 当它们是软的，几乎不带参数，只有一层深度
- 当存在相当数量的重复（确切的数量是主观的）

一个测试怪癖的例子是下面这个

```ts
/**
 * Clear a Console's textarea.
 * Work around cy.clear sometimes not working in the Console's textareas.
 */
Cypress.Commands.add('clearConsoleTextarea', { prevSubject: 'element' }, el => {
  cy.wrap(el).type('{selectall}', { force: true }).trigger('keydown', {
    keyCode: 46,
    which: 46,
    force: true,
  });
});
```

我创建了中心的 `cy.clearConsoleTextarea`，原因如下：

1. 这是一种权宜之计 😊
2. 对于新手来说，阅读 `trigger('keydown')` 而不是使用更符合习惯的 `cy.clear` 是有点奇怪的，我不想在每个地方都留下解释的注释。
3. 该命令由 5 行代码组成，将使测试代码变得过长而毫无必要。 

以下内容是软抽象的一个例子：

```ts
function expectSuccessNotification = (title: string) {
  cy.get('.notification-success')
    .should('be.visible')
    .should('contain', title)
}
```

我喜欢它的原因是

1. 它不依赖其他抽象代码：如果我的测试在 `expectSuccessNotification('Table created!')` 失败，我不必陷入深奥的代码中，理解 `expectSuccessNotification` 背后发生了什么。
2. 它只接受一个变量，而不是很多选项；也没有包含那些在理解代码最终执行内容时变得复杂的条件。
3. **它专注于特定用例**。它不试图一次性涵盖所有通知类型、内容等。其他专注于特定用例的函数会处理。
4. 如果你重构通知系统，你有一个中心点进行重构，以适应新的通知系统。

相反，这是我不希望拥有的（在谈论通知工具时）。

```ts
export const expectNotification = (
  {
    type,
    title,
    message,
  }: {
    type: 'success' | 'error';
    title: string;
    message?: string;
  },

  timeout = 10000
) => {
  const el = cy.get(
    type === 'success' ? '.notification-success' : '.notification-error',
    { timeout }
  );

  el.should('be.visible');
  el.should('contain', title);

  if (message) el.should('contain', message);
};
```

我对上面的例子不太喜欢，原因有两点：

1. 它试图一次性涵盖太多用例。
2. 如果测试失败，你必须处理让整个体验变成噩梦的各种条件。

在[Hasura 控制台 UI 编码模式：测试](https://dev.to/noriste/hasura-console-ui-coding-patterns-testing-281d)文章中，你可以找到我们在内部遵循的更多最佳实践。

## 匹配测试代码和测试运行器命令

![测试代码与 Cypress 面板并排显示，带有一些红色箭头来匹配代码和 Cypress 日志](../../assets/images/test-code-with-debugging-in-mind/no-match-between-code-and-runner.jpg)

Cypress 测试运行器有助于理解应用程序中发生了什么以及执行了哪些命令，但在调试测试时，很难立即在测试运行器和代码之间建立关联。而且，日志无法帮助理解测试从功能角度正在做什么（例如，日志说“在文本区域中键入”，但没有说明“在类型配置文本区域中键入”）。因此，查找失败的根本原因是困难的。Cypress 会为失败的测试记录视频，但如果阅读者/调试者不能在日志和测试在普通英语中所做的事情之间建立直接关联，则视频就毫无用处。

### 请看下面的内容

![代码和 Cypress 面板并排显示，有许多自定义日志，可以直接将 Cypress 中发生的情况与代码中的特定点连接起来。](../../assets/images/test-code-with-debugging-in-mind/match-between-code-and-runner.jpg)

我添加了一个日志，报告测试正在进行的操作，使测试代码与测试运行程序之间能够直接对应。 (`cy.log('**--- Type in the Webhook Handler field**');`).

请注意，你可以向 'cy.log' 传递更多参数，这些参数将在单击记录的命令时直接显示在开发工具的控制台中。

![Cypress 测试运行器展示了已记录对象的值。](../../assets/images/test-code-with-debugging-in-mind/cy-log-console.jpg)

Storybook 和 Playwright 已经引入了`step`实用工具的概念，可以用英语解释测试中的步骤。Cypress 没有相同的选项，因此我认为我提出的`cy.log`是很有价值的。

这里需要注意：不要将`cy.log`链在一起，因为它不是一个查询命令，不会对链进行重试。

截至 Cypress V12 版本，`cy.log`在函数级别不进行重试。

例如：

```js
cy.log('foo').get('bar').should('baz') // does not retry
cy.get('bar').should('baz') // retries the whole chain until the assertion passes (you have 10 sec timeout set)
```

值得注意的是，即使 Cypress 没有 `step`，[Filip Hric](https://github.com/filiphric) 的 [cypress-plugin-steps](https://github.com/NoriSte/ui-testing-best-practices/issues/43) 也是一个有效的替代选择。

## 使用清晰的选择器

看一下这段代码

```ts
cy.get('textarea')
  .eq(0)
  .type(`{enter}{uparrow}${statements.createMutationGQLQuery}`, {
    force: true,
  });
```

`cy.get('textarea').eq(0)` 是什么？在没有更好的选择器的情况下，我建议将它们放在 Cypress 的别名下，比如

```ts
// Assign an alias to the most unclear selectors for future references
cy.get('textarea').eq(0).as('actionDefinitionTextarea');
cy.get('textarea').eq(1).as('typeConfigurationTextarea');
```

然后通过这种方式来引用它们

```ts
cy.get('@actionDefinitionTextarea').clearConsoleTextarea().type(/* ... */);
```

以提高读者的体验。

## 减少 data-testid 属性

我不想讨论测试本身及其对测试结果可信度的价值，只想谈谈 data-testid 属性在调试阶段的影响。

如果无法从页面中检索带有 data-testid 属性的元素，可能的问题有：

1. 元素不存在。
2. 元素存在，但它没有该属性。
3. 元素存在，具有该属性，但值不符合预期。

上述所有问题都会**导致开发人员重新启动测试、检查元素、查找与测试相关的属性等**。相反，如果测试基于文本内容，仅通过截图就足以了解测试搜索的文本是否不存在或错误。

此外，对于那些必须处理 data-testid 的工程师来说，还有一些不足之处：

1. 在重构期间必须维护与测试相关的属性，但在有数百个属性时并不容易。
2. 如果测试相关的属性在页面上是唯一的，那么它们会很有帮助。然而，当你有数百个这样的属性时，很难保证它们是唯一的。

我的建议是仅在以下情况使用 data-testid 属性：

- 用于节，而不是元素（例如 Header、Footer 等），以减小基于文本搜索的范围。以下是一个示例：

```ts
cy.get('[data-test="Actions list"]').within(() => { // <-- reduce the scope
  cy.contains('login') // <-- the "login" text could exist more times in the page
})
```

- 非文本元素，如图标、图片等。

最后但同样重要的是：我建议为它们赋予用户友好的值，而不是采用程序员的命名风格（例如，“操作列表”而不是“actionsList”），尤其是当该部分显示相同文本时。这样可以直接关联测试代码、Cypress 的测试运行器和页面的文本内容。

## 将相关操作分组

通常来说，阅读一系列平面的交互并不能帮助理解测试运行的页面结构。

例如：

- 获取 1 并点击
- 获取 2 并点击
- 获取 3 并点击
- 获取 4 并点击
- 获取 5 并点击
- 获取 6 并点击
- 获取 7 并点击
- 获取 8 并点击

然而，将列表展开可以帮助读者构建一个有关所涉及部分位置的心理模型

- 在块 1 内
  - 获取 1 并点击
  - 获取 2 并点击
  - 获取 3 并点击
- 在块 2 内
  - 获取 4 并点击
  - 获取 5 并点击
  - 获取 6 并点击
- 获取 7 并点击
- 获取 8 并点击

![Cypress 界面展示 cy.within](../../assets/images/test-code-with-debugging-in-mind/cy-within.png)

再强调一下：Storybook 和 Playwright 已经引入了“步骤（step）”实用程序的概念，该实用程序可以将操作进行分组，而上述建议在 Cypress 中非常实用。

## 相关章节

- 🔗 [从晦涩难懂的 React 组件测试到简单、易读的版本](/sections//real-life-examples/from-unreadable-react-component-tests-to-simple-ones.zh.md)

<br/><br/>

*由 [NoriSte](https://github.com/NoriSte) 在 [dev.to](https://dev.to/noriste/improving-ui-testss-code-to-ease-debugging-them-later-2478j)进行发表.*
