# [过时] Cypress + Storybook。保持测试场景、数据和组件渲染在一个地方

*此部分现在被标记为过时，因为它涉及到非常陈旧的 Cypress 和 Storybook 版本（它们现在都已完全支持组件测试）。*

---

_Russian version: [Cypress + Storybook. Хранение тестового сценария, данных и рендеринг компонента в одном месте](https://habr.com/ru/post/497544/)._

## 一段简要说明

许多人选择使用 Cypress 来测试通过 Storybook/Styleguidist/Docz 托管的组件。[@NoriSte](./cypress-and-storybook.zh.md) 的示例建议在 Storybook 中创建一些故事，将组件放在其中，并将重要数据暴露给全局变量，以便在 Cypress 中以任何你喜欢的方式进行测试。这是一个很好的方法，但测试被分割成了 Storybook 和 Cypress 之间的几个部分。

在这里，我想展示如何更进一步，充分利用在 Cypress 中执行 JavaScript 的优势。要亲自体验，请[下载源代码](https://github.com/daedalius/article-exposing-component-from-storybook-to-handle-them-in-cypress)，然后在控制台中执行 `npm i` 和 `npm test`。

- 你可以从 Storybook Story 中公开组件引用，在 Cypress 中以任何你喜欢的方式进行测试，而无需将测试逻辑拆分成多个部分。
- 对于我们的团队来说，Cypress 显得非常强大，因此我们没有其他实用程序使用 js-dom 在底层测试 UI 组件。

## 任务

假设我们正在为现有的日期选择器组件编写适配器，以便在所有公司网站上使用。我们不希望意外破坏任何东西，因此我们必须通过测试来覆盖它。

## Storybook

从 Storybook 需要的一切是一个保存对测试组件引用的空 Story。为了不那么无用，这个 Story 渲染了单个 DOM 节点。这个节点将是我们测试中的战区。

```jsx
import React from 'react'
import Datepicker from './Datepicker.jsx'

export default {
  component: Datepicker,
  title: 'Datepicker',
}

export const emptyStory = () => {
  // Reference to retrieve it in Cypress during the test
  window.Datepicker = Datepicker

  // Just a mount point
  return <div id="component-test-mount-point"></div>
}
```

好的，我们已经在 Storybook 中完成了操作。现在让我们转向 Cypress。

## Cypress

个人而言，我喜欢从列举测试用例开始。我们似乎有以下测试结构：

```jsx
/// <reference types="cypress" />

import React from 'react'
import ReactDOM from 'react-dom'

/**
 * <Datepicker />
 * * renders text field.
 * * renders desired placeholder text.
 * * renders chosen date.
 * * opens calendar after clicking on text field.
 */

context('<Datepicker />', () => {
  it('renders text field.', () => {})

  it('renders desired placeholder text.', () => {})

  it('renders chosen date.', () => {})

  it('opens calendar after clicking on text field.', () => {})
})
```

好的。我们需要在任何环境中运行这个测试。打开 Storybook，通过点击侧边栏中的 "在新标签页中打开画布 Open canvas in new tab" 按钮直接进入空的 Story。复制该 URL 并使用 Cypress 访问：

```jsx
const rootToMountSelector = '#component-test-mount-point'

before(() => {
  cy.visit('http://localhost:12345/iframe.html?id=datepicker--empty-story')
  cy.get(rootToMountSelector)
})
```

正如你可能猜到的那样，为了进行测试，我们将在具有 `id=component-test-mount-point` 的相同`<div>` 中渲染所有组件状态。为了确保测试之间不相互影响，我们必须在下一次测试执行之前卸载此处的任何组件。让我们添加一些清理代码：

```jsx
afterEach(() => {
  cy.document().then((doc) => {
    ReactDOM.unmountComponentAtNode(doc.querySelector(rootToMountSelector))
  })
})
```

现在我们准备完成测试。检索组件引用，渲染组件并做一些断言：

```jsx
const selectors = {
  innerInput: '.react-datepicker__input-container input',
}

it('renders text field.', () => {
  cy.window().then((win) => {
    ReactDOM.render(<win.Datepicker />, win.document.querySelector(rootToMountSelector))
  })

  cy.get(selectors.innerInput).should('be.visible')
})
```

你发现了吗？没有任何阻碍我们直接将任何 props 或数据传递给组件！一切都在一个地方 - 在 Cypress 中！

## 使用包装器进行逐步测试

有时候，我们希望测试组件根据变化的 props 以可预测的方式表现。考虑具有 `showed` props 的 `<Popup />` 组件。当 `showed` 为 `true` 时，`<Popup />` 是可见的。然后，将 `showed` 更改为 `false`，`<Popup />` 应该变为隐藏。如何测试这种转换呢？

这些问题在命令式的方式中很容易处理，但在声明式的 React 中，我们需要想出一些方法。在我们的团队中，我们使用了一个具有状态的额外包装组件来处理它。这里的状态是布尔值，它响应于 "showed" props。

```jsx
let setPopupTestWrapperState = null
const PopupTestWrapper = ({ showed, win }) => {
  const [isShown, setState] = React.useState(showed)
  setPopupTestWrapperState = setState
  return <win.Popup showed={isShown} />
}
```

现在我们即将完成测试：

```jsx
it('becomes hidden after being shown when showed=false passed.', () => {
  // arrange
  cy.window().then((win) => {
    // initial state - popup is visible
    ReactDOM.render(
      <PopupTestWrapper showed={true} win={win} />,
      win.document.querySelector(rootToMountSelector)
    )
  })

  // act
  cy.then(() => {
    setPopupTestWrapperState(false)
  })

  // assert
  cy.get(selectors.popupWindow).should('not.be.visible')
})
```

> 提示：如果这个钩子无法正常工作，或者你不喜欢在组件外部调用这个钩子 - 通过简单的类重写包装器。

## 测试组件方法

实际上，我从未编写过这样的测试。这个想法是在写这篇文章的过程中浮现的。可能在单元测试风格中测试组件会很有用。

然而，你可以很容易地在 Cypress 中实现这一点。只需在渲染之前为组件创建一个 ref。值得一提的是，ref 可以访问组件的状态和其他元素。

我给 `<Popup \>` 添加了 `hide` 方法，它可以强制将其隐藏（举例而已）。以下测试看起来是这样的：

```jsx
it('closes via method call.', () => {
  // arrange
  let popup = React.createRef()
  cy.window().then((win) => {
    // initial state - popup is visible
    ReactDOM.render(
      <win.Popup showed={true} ref={popup} />,
      win.document.querySelector(rootToMountSelector)
    )
  })

  // act
  cy.then(() => {
    popup.current.hide()
  })

  // assert
  cy.get(selectors.popupWindow).should('not.be.visible')
})
```

## 总结：每个参与者的角色

Storybook：

- 托管包含用于测试目的的打包 React 组件的 Storybook Stories。
- 提供一个真实的非合成环境来运行测试。
- 每个 Story 在全局变量中暴露一个组件（以便稍后在 Cypress 中检索它）。
- 每个 Story 暴露一个组件挂载点（用于在测试中挂载组件）。
- 能够在新标签页中以隔离的方式打开每个组件。

> 专业提示：请为组件库或页面运行另一个 Storybook 实例。

Cypress：

- 包含并运行测试和 JavaScript。
- 访问隔离的组件 Stories，从全局变量中检索组件引用。
- 根据测试需求呈现组件（使用任何数据或测试条件，例如移动分辨率）。
- 为你提供非常方便的 UI，以便查看测试的执行情况。

## 结论

在这里，我想表达一下我的个人观点和同事们对可能在阅读过程中出现的问题的看法。下面所写并不自称为真实，可能与实际情况不同，并且可能包含错误。

### 我的测试工具在幕后使用 js-dom。这样做会限制我吗？

- Js-dom 是一个合成环境。分离的 DOM 不是一个真正的浏览器。
- 以 js-dom 与用户一样进行交互实际上并不起作用。特别是涉及模拟输入事件时。
- 如果一个组件因为一个不正确的 z-index 而在 CSS 中被破坏，那么从编写的单元测试中能得到多少信心呢？如果组件由 Cypress 进行测试，你将看到一个错误。
- 你是盲目地编写单元测试。但是为什么呢？

### 我应该选择建议的方法吗？

- 如果你将测试用作开发环境 - 那当然是肯定的！
- 如果你将测试视为**活**文档 - 是的。
- 如果你真的是为了覆盖与实现和 React 生命周期过于接近的事物而编写单元测试 - ... 我不知道。我很久没有编写这样的测试了。你确定所覆盖的逻辑是组件的责任吗？也许该逻辑应该被提取并相应地进行测试？

### 为什么不使用 cypress-react-unit-test 呢？为什么我们需要 Storybook？

也许我们未来是为了测试组件。将不再需要维护 Storybook 的单独实例，所有测试将完全由 Cypress 负责，配置将简化等等。
但是现在这个工具存在一些问题，使其提供的环境无法完全运行测试。希望 Gleb Bahmutov 和 Cypress 团队能够解决这些问题 🤞

_由 [daedalius](https://github.com/daedalius) 在 [Medium](https://itnext.io/cypress-storybook-keeping-test-scenario-data-and-component-rendering-in-one-place-c57b23cc1640) 和 [habr.com（俄文）](https://habr.com/ru/post/497544/) 上进行联合发布。_