# [过时] 使用 Cypress 和 Storybook 进行组件测试

*此部分已被标记为过时，因为它涉及到 Cypress 和 Storybook 的非常旧版本（它们现在都已完全支持组件测试）*

---

_*更新：在这个实验性方法之后，请查看“[使用 Cypress 进行 React 组件单元测试](./cypress-react-component-test.zh.md)”章节，随着 Cypress 4.5.0 的发布，一切都变得更加简化和有效！*_

_*更新 2：[Cypress 7 推出了全新的组件测试](https://docs.cypress.io/guides/component-testing/introduction#What-is-Component-Testing)支持，不要错过！还有其他令人振奋的消息，要感谢 Storybook 6.2 版本的发布！*_

## 为什么要孤立地测试组件？

组件是你应用程序的构建模块，Storybook 允许你在孤立的环境中构建它们，**检查**它们是否正常工作，并且是否与图形布局对齐，与团队共享等。基本上，Storybook 执行两种类型的组件检查：

- 视觉测试：使用[Percy](https://percy.io/)或[Applitools](https://applitools.com/)运行，两者都可以轻松集成到 Storybook 中以自动执行这些检查

- 功能测试：在组件故事中手动执行，为什么不也自动化呢？本章讨论了这个问题

## 组件测试工具

当前前端测试趋势有两个赢家：**[DOM 测试库](https://testing-library.com)** 和 **[Cypress](https://www.cypress.io/)**。它们是两个完全不同的工具，但在某些方面它们**重叠**。DOM 测试库明确用于通过第三方测试运行器（如[Jest](https://jestjs.io/)）正确且快速地测试组件，非常适合这样的场景。Cypress 则旨在以简单可靠的方式自动化浏览器。

为什么它们会有交集呢？因为从技术上讲，你可以使用 DOM 测试库测试整个应用程序，或者可以使用 Cypress 测试单个组件。

使用 **DOM 测试库** 测试整个应用程序：

- 优点：速度**非常快**

- 缺点：它渲染 HTML，但 HTML 不是由真实浏览器渲染的，因此**CSS 部分不起作用**

- 缺点：读取巨大的 HTML 而不是在浏览器中消耗结果很麻烦

- 缺点：第三方组件可能无法与 [jsdom](https://github.com/jsdom/jsdom) 环境完全兼容，或者在由 DOM 测试库渲染时无法按预期工作

使用 **Cypress** 测试组件：

- 优点：它自动化了一个**真实的浏览器**，组件在将要使用的相同环境中进行了测试

- 优点：你可以测试用户将要看到/使用的组件的方式

- 优点：你可以检查你的**Storybook 是否正常工作**。你的故事可能存在错误，因此拥有一些工作正常的组件并不能保证 Storybook 正常工作。而团队的其余成员可能将 Storybook 视为团队/公司的组件库。

- 缺点：**它需要一个运行的主机**/网站，允许你与组件本身进行交互（如 Storybook）

- 缺点：测试数百个组件可能会很慢

请注意：对于 Cypress，有 [Cypress Testing Library](https://testing-library.com/docs/cypress-testing-library/intro) 插件，它允许你利用 DOM 测试库的相同的 _findByText_、_findByPlaceholderText_、_findByTestId_ 等 API。我喜欢它，我总是使用它，但它们在 Cypress 内部工作，而不是在其他测试运行器中工作

## 一个真实的例子：虚拟列表

最近，我开发了一个 VirtualList 组件，并使用它来检查 Storybook 和 Cypress 如何一起工作。VirtualList 是一个只渲染可见项以确保最高性能的列表。查看 [React Virtualized](https://bvaughn.github.io/react-virtualized/#/components/List) 以了解其工作原理。除了虚拟化之外，我的 VirtualList 还处理项目的点击和项目的多选。

除了大

量的单元测试外，我想运行一些组件测试，但是如上所述，第三方库（比如我们使用的 [React Smooth Scrollbar](https://www.npmjs.com/package/react-smooth-scrollbar)）可能与 [React Testing Library](https://testing-library.com/docs/react-testing-library/intro)（DOM 测试库的 React 版本）不兼容，因此 Cypress 是唯一的选择。

## 测试挑战

使用 Cypress 在 Storybook 中测试组件而不是运行标准的 E2E 测试有哪些最大的挑战？与使用 [React Testing Library](https://testing-library.com/docs/react-testing-library/intro)（DOM 测试库的 React 版本）进行标准组件测试相比，有什么挑战？

### 数据源控制

使用 Cypress，我们习惯于加载整个应用程序，通过静态固件（在 [UI 集成测试](../testing-strategy/component-vs-integration-vs-e2e-testing.zh.md) 的情况下）或读取/拦截后端数据（在 [E2E 测试](../testing-strategy/component-vs-integration-vs-e2e-testing.zh.md) 的情况下）来控制数据，并使用此数据来断言 UI 是否正常工作。因此，我们可以控制，或者至少了解，UI 将要使用的数据。**不了解数据将意味着没有断言**。
但在（无状态）组件故事中，数据直接由代表故事并渲染组件的组件传递给组件。为了了解数据并对由故事组件显示的内容进行适当的断言，**数据需要由故事暴露并由 Cypress 读取**。

请注意：如果 Cypress 测试与故事的组件位于同一个存储库中，你可以从故事文件中导入数据，但下面显示的示例来自不在故事存储库中的测试。

### 隐藏内容

**虚拟列表会从 DOM 中删除不可见的组件**。这对测试意味着什么？我们不能指望所有元素都存在于 HTML 中的事实上。无论是 React Testing Library 还是 Cypress 都不能完全利用它们来理解虚拟列表的工作方式，因为一旦需要断言一个在可见区域之外的项的存在，就无法利用通常的 _cy.contains_ / _findByText 等_ 实用工具：因为这些项根本不存在。

VirtualList 是一个受控组件，因此它会通知父组件有哪些被渲染的项以及哪些是选中的项。在标准的 [组件测试](../testing-strategy/component-vs-integration-vs-e2e-testing.zh.md) 中，我们可以断言传递给 VirtualList 组件的上下文和下行的 props，故事直接控制这些 props，但是再次，Cypress 需要读取这些数据，因此故事必须将其暴露出来。

### 我们是在测试故事还是组件？

区别可能微妙。由于 VirtualList 是一个受控组件，故事的组件对其进行控制。故事组件内部的控制逻辑是需要测试的吗？控制组件和被控制组件是否可分离？可能不太可能。

标准测试有两个角色——测试运行器和受控组件——而使用 Cypress 与 Storybook 时有**三个角色——测试运行器、故事和受控组件**——。因此，如果组件测试控制组件并直接访问回调数据，Cypress 需要故事组件正常工作。

### 滚动和渲染项控制

VirtualList 组件利用惯性滚动，并且由于故事添加了成千上万个要渲染的项以完全显示 VirtualList 的潜力，所以滚动 VirtualList 可能会导致测试和下一个测试之间的渲染项略有不同。因此，我认为我们不能提前知道在滚动列表后将渲染哪些项（例如从第 100 个到第 110 个，或从第 101 个到第 111 个）。

为什么？因为**我讨厌基于脆弱假设的易碎测试**。测试不应该设计成可以抵御每一种可能的变化，而是必须足够健壮，能够经受小的变化。对于 VirtualList 测试来说，这意味着一旦滚动，将直接从 HTML 中检索渲染的项。我们将在后面深入探讨。

## 请给出代码

我们要编写的第一个测试需要检查，如果列表收到了 10000 个项目，只渲染其中的一小部分。这是 Virtual List 的基本功能。

首先是 RenderItem 组件，它只是交替更改颜色以直观标识行

```js
// every `item` is an { id: string, name: string}
const getItemText = (item) => `id: ${item.id} - ${item.name}`

const RenderItem = ({ item }) => {
  return (
    <div
      style={{
        height: '30px',
        backgroundColor: parseInt(item.id) % 2 ? '#FAFAFA' : '#EEE',
      }}
    >
      {getItemText(item)}
    </div>
  )
}
```

我们将处理的原始故事如下

```js
// `getStoryItems` allows to create a high amount of items, it's used by every story
export const With10000Items = () => {
  return (
    <>
      <h4>The scroll must be fluid</h4>
      <VirtualList
        items={getStoryItems({ amount: 10000 })}
        getItemHeights={() => 30}
        RenderItem={RenderItem}
        listHeight={300}
      />
    </>
  )
}
With10000Items.story = {
  name: 'With 10000 items',
}
```

我们需要调整故事以全局公开以下内容：

- `items` 数组：Cypress 测试需要知道用于填充列表的数据

- `getItemText` 函数：这样 Cypress 测试就无需关心如何从项目获取渲染的文本。Cypress 需要将项目转换为文本，以从其文本中检索渲染的项目，避免使用 `data-test` 属性

- 可见项目的数量：Cypress 测试需要断言哪些项目被渲染，哪些没有

以下是经过更新以通过全局 `storyData` 变量公开变量的故事：

```js
export const With10000Items = () => {
  const itemHeight = 30
  const listHeight = 300
  const items = React.useMemo(() => getStoryItems({ amount: 10000 }), [])

  // exposing data for Cypress
  React.useEffect(() => {
    // global is `window`
    global.storyData = {
      items,
      visibleItemsAmount: Math.ceil(listHeight / itemHeight),
      getItemText,
    }
  }, [items])

  return (
    <>
      <h4>The scroll must be fluid</h4>
      <VirtualList
        items={items}
        getItemHeights={() => itemHeight}
        RenderItem={RenderItem}
        listHeight={listHeight}
      />
    </>
  )
}
With10000Items.story = {
  name: 'With 10000 items',
}
```

### Cypress 的测试

测试将执行以下操作：

- 访问 Storybook 页面

- 收集公开的数据

- 检查渲染的项目

第一和第二点的代码如下：

```js
it('When the component receives 10000 items, then only the minimum number of items are rendered', () => {
  cy.visit('/iframe.html?id=virtuallist--with-10000-items')

  cy.window()
    .its('storyData')
    .should((storyData) => {
      // the story must expose some variables
      expect(storyData.items).to.be.to.have.length(10000)
      expect(storyData.visibleItemsAmount).to.be.greaterThan(0)
      expect(storyData.getItemText).to.be.a('function')
    })
    .then(() => {
      // the test code
    })
})
```

为什么要断言公开的 `storyData`？因为错误的 `storyData` 会导致测试失败，如果测试失败，它必须直接引导我们找到问题。如果组件渲染因为错误的 `storyData` 而失败，我们的测试根本不应该运行。这可以被视为**与数据相关的冒烟测试**。

现在，测试的缺失部分：检查哪些项目被渲染，哪些没有被渲染。

```js
it('When the component receives 10000 items, then only the minimum number of items are rendered', () => {
  cy.visit('/iframe.html?id=virtuallist--with-10000-items')

  cy.window()
    .its('storyData')
    .should((storyData) => {
      // the story must expose some variables
      expect(storyData.items).to.be.to.have.length(10000)
      expect(storyData.visibleItemsAmount).to.be.greaterThan(0)
      expect(storyData.getItemText).to.be.a('function')
    })
    .then(({ visibleItemsAmount, getItemText, items }) => {
      // items visibility check
      const visibleItems = items.slice(0, visibleItemsAmount - 1)
      visibleItems.forEach((item) => {
        cy.findByText(getItemText(item)).should('be.visible')
      })

      // first not-rendered item check
      cy.findByText(getItemText(items[visibleItemsAmount])).should('not.exist')
    })
})
```

- 我们使用公开的 `visibleItemsAmount` 变量来仅检索已渲染的项目

  const visibleItems = items.slice(0, visibleItemsAmount - 1)

- 我们使用公开的 `getItemText` 函数从页面中检索每个项目

  cy.findByText(getItemText(item))

- 我们断言所有预期可见的项目

  cy.findByText(getItemText(item)).should('be.visible')

- 我们断言下一个项目在页面中不存在

```js
cy.findByText(getItemText(items[visibleItemsAmount])).should('not.exist')
```

请注意，`cy.findByText` Cypress 命令来源于… [Cypress Testing Library](https://testing-library.com/docs/cypress-testing-library/intro) 😊，这是我最喜欢的检索页面元素的方式，因为它的行为就像用户一样：通过文本内容读取/检索元素。我使用它而不是 `cy.contains`，但它们执行相同的功能。

这就是结果：

![Cypress 受控浏览器，左侧是所有断言结果，右侧是 Storybook 故事。](../../assets/images/cypress-storyboook/list-test.png '第一个测试的结果：加载的故事和所有断言的结果。')_第一个测试的结果：加载的故事和所有断言的结果。_

这个测试足以检查虚拟列表是否只渲染了最小数量的项目。

最初，我试图检查在虚拟列表滚动时浏览器是否以 60 FPS 运行。为什么？因为**UI 测试必须检查用户所见的内容，以与用户相同的方式使用 UI**。从用户的角度来看，虚拟列表的唯一目标是必须流畅运行，无论列表中包含多少项目。但是可靠地测量 FPS 难度很大，因为测量结果会受到以下影响：

- 您使用的 Cypress 命令：每个 Cypress 动作都会减慢浏览器的速度

- 用于运行测试的机器（或 Docker 镜像）的资源数量

由于不可能获得可靠的计数 — 一些初始值将被丢弃等等 — 我决定仅检查渲染的项目数量。如果我确信只有 10000 项中的 10 项被渲染，那么我确信列表运行流畅。总的来说：记住**脆弱的测试比缺失的测试更糟糕**。

## 第二个测试：滚动

首先要说明：VirtualList 组件使用 [Smooth Scrollbar](https://idiotwu.github.io/smooth-scrollbar/)，Smooth Scrollbar 已经有了自己的测试，因此我们主要关注的是 VirtualList 组件如何响应滚动。在使用**第三方库**时，你不需要测试该库是否有效。该库**应该有自己的测试**，如果没有，就换个库吧！千万别为第三方库编写测试。

滚动测试的步骤包括：

- 触发滚动

- 等待滚动结束

- 检查渲染的项

因此，触发滚动的测试代码如下

```js
// triggers the wheel event
cy.findByTestId('VirtualList').trigger('wheel', {
  deltaX: 0,
  deltaY: 1000,
})
```

请注意，`VirtusList` 组件应该在渲染时生成一个带有 `data-test="VirtualList"` 属性的 DOM 元素。其余的由 Cypress 处理，`trigger` 直接映射了 [jQuery trigger API](https://api.jquery.com/trigger/)（Cypress 利用 jQuery 来尽量缩短测试代码）。再次强调一下：Cypress 的本地等价于 `cy.findByTestId('VirtualList')` 是 `cy.get('[data-testid=VirtualList]')`。

至于“等待滚动结束”的部分，我们将使用 [Cypress waitUntil](https://github.com/NoriSte/cypress-wait-until) 插件。为什么我们应该使用 waitUntil 插件而不是等待固定时间的原因在 [等待，不要让你的 E2E 测试休眠](../generic-best-practices/await-dont-sleep.zh.md) 章节中有详细的解释。
自定义等待的代码如下：

```js
// waits until the scrollbar handle stops
let scrollbarHandleY = Number.NEGATIVE_INFINITY
cy.get('.scrollbar-thumb-y').waitUntil(
  ($scrollbarHandle) => {
    const [newY, previousY] = [$scrollbarHandle.offset().top, scrollbarHandleY]
    scrollbarHandleY = newY
    return previousY === newY
  },
  {
    customMessage: 'The inertial scroll ends',
  }
)
```

因此，我们确保测试等待了正确的时间。

### 检查渲染的项目

这意味着：

- 查找第一个已渲染的项目

- 检查第一个已渲染的项目是否不是第一个项目。虚拟列表开始渲染从第 0 到 10 个项目。一旦滚动，它必须已渲染第 X 到 X+10 个项目。我们并不知道 X 的具体值，也并不关心，否则测试将与确切的已渲染项目绑定在一起。第一个已渲染的项目是第 60 或第 61 个并不重要。检查确切的已渲染项目超出了这个通用滚动测试的范围。测试的稳定性对我们有利。

- 一旦找到第一个已渲染的项目，我们检查接下来的十个项目是否都已渲染

我们将逐步进行每个步骤，测试的最终代码如下：

```js
it('When the component is scrolled, then the rendered items are not the first ones', () => {
  cy.visit('/iframe.html?id=virtuallist--with-10000-items')

  cy.window()
    .its('storyData')
    .should((storyData) => {
      // the story must expose some variables
      expect(storyData.items).to.be.to.have.length(10000)
      expect(storyData.visibleItemsAmount).to.be.greaterThan(0)
      expect(storyData.getItemText).to.be.a('function')
    })
    .then(({ visibleItemsAmount, getItemText, items }) => {
      // triggers the wheel event
      cy.findByTestId('VirtualList').trigger('wheel', {
        deltaX: 0,
        deltaY: 1000,
      })

      // waits until the scrollbar handle stops
      let scrollbarHandleY = Number.NEGATIVE_INFINITY
      cy.get('.scrollbar-thumb-y')
        .waitUntil(
          ($scrollbarHandle) => {
            const [newY, previousY] = [$scrollbarHandle.offset().top, scrollbarHandleY]
            scrollbarHandleY = newY
            return previousY === newY
          },
          {
            customMessage: 'The inertial scroll ends',
          }
        )
        // (manually) looks for the first rendered element
        .then(() =>
          items.findIndex((item) => !!Cypress.$(`*:contains("${getItemText(item)}")`).length)
        )
        // checks that the rendered items are not the initially rendered ones
        .should('be.greaterThan', 10)
        // checks the rendered items
        .then((firstVisibleItemIndex) => {
          const visibleItems = items.slice(
            firstVisibleItemIndex,
            firstVisibleItemIndex + visibleItemsAmount - 1
          )
          visibleItems.forEach((item) => cy.findByText(getItemText(item)).should('be.visible'))
        })
    })
})
```

逐步进行：检索第一个已渲染项目的代码如下：

```js
items.findIndex((item) => !!Cypress.$(`*:contains("${getItemText(item)}")`).length)
```

为什么不使用 `cy.findByText` 命令来检索呢？因为在 Cypress 中，每个 `cy.get` 命令（也是 `cy.findByText` 命令核心的一部分）都内置了断言，[在官方文档中查看](https://docs.cypress.io/api/commands/get.html#Assertions)。基本上，如果页面上不存在该元素，`cy.get` 将导致测试失败。但由于我们需要浏览未渲染的项目，直到找到第一个已渲染的项目，我们应该手动进行。Cypress.\$ 是 jQuery 的全局实例，`cy.findByText("XXX")` 的 jQuery 版本是 `Cypress.$(`\*:contains("XXX")`)`，而且 jQuery 版本如下：

```js
cy.findByText('XXX').should('exist')
```

是

```js
!!Cypress.$(`*:contains("XXX")`).length
```

唯一的区别是如果元素不存在，它不会导致失败。正如前面所说：我习惯使用 `cy.getByText`，但本地的 `cy.contains` 也能达到同样的效果！

一旦找到第一个已渲染项目的索引，我们只需检查接下来的已渲染项目。

```js
.then(() =>
  items.findIndex(
    item => !!Cypress.$(`*:contains("${getItemText(item)}")`).length,
  )
)
// checks that the rendered items are not the initially rendered ones
.should('be.greaterThan', 10)
// checks the rendered items
.then(firstVisibleItemIndex => {
  const visibleItems = items.slice(
    firstVisibleItemIndex,
    firstVisibleItemIndex + visibleItemsAmount - 1,
  )
  visibleItems.forEach(item =>
    cy.findByText(getItemText(item)).should('be.visible'),
  )
})
```

[这是测试的录像视频](https://www.youtube.com/watch?v=OEMkz_86bqY)。

请注意，我们不关心最后半个可见的项目。测试的范围是检查已渲染的项目是否不是第一个，并且至少有十个项目已被渲染，不需要检查第 11 个项目。

我们可以将测试的滚动部分移到一个独立的实用程序，类似于：

```js
const scrollVirtualList = ($list, deltaY = 1000) => {
  cy.wrap($list)
    .trigger('wheel', {
      deltaX: 0,
      deltaY,
    })
    .within(() => {
      // waits for the inertial scroll end
      let scrollbarY = Number.NEGATIVE_INFINITY
      getScrollbar().waitUntil(
        ($scrollbar) => {
          const newY = $scrollbar.offset().top
          const previousY = scrollbarY
          scrollbarY = newY
          return previousY === newY
        },
        {
          customMessage: 'The inertial scroll end',
        }
      )
    })
}
```

而且我们可以对查找第一个已渲染项目的过程采用相同的方法。

```js
const getFirstRenderedItemIndex = (items, getItemText) => {
  return items.findIndex((item) => !!Cypress.$(`*:contains("${getItemText(item)}")`).length)
}
```

这种分离的方式有助于测试的可读性，看一下：

```js
it('When the component is scrolled, then the rendered items are not the first ones', () => {
  cy.visit('/iframe.html?id=virtuallist--with-10000-items')

  cy.window()
    .its('storyData')
    .should((storyData) => {
      // the story must expose some variables
      expect(storyData.items).to.be.to.have.length(10000)
      expect(storyData.visibleItemsAmount).to.be.greaterThan(0)
      expect(storyData.getItemText).to.be.a('function')
    })
    .then(({ visibleItemsAmount, getItemText, items }) => {
      cy.findByTestId('VirtualList')
        // we leverage the new `scrollVirtualList` function
        .then(scrollVirtualList)
        .then(() => getFirstRenderedItemIndex(items, getItemText))
        .should('be.greaterThan', 10)
        .then((firstVisibleItemIndex) => {
          const visibleItems = items.slice(
            firstVisibleItemIndex,
            firstVisibleItemIndex + visibleItemsAmount - 1
          )
          visibleItems.forEach((item) => cy.findByText(getItemText(item)).should('be.visible'))
        })
    })
})
```

## 第三个测试：选择

VirtualList 组件通过键盘修饰键支持选择和**多项选择**。我们只需点击项目，用键盘修饰键点击项目，并检查哪些项目被选中。这些操作有什么难点呢？

- 首先：什么是“已选择”？从用户的角度来看，已选择的项目是“突出显示”的项目，但检查元素的样式并不可靠。我们可以为组件添加一个 `data-selected` 属性…

- … 但存在一个结构性的问题：项目可能根本没有被渲染！如果我们点击第一个项目（从第 1 个到第 10 个项目可见），然后滚动大约 20 个项目（从第 21 个到第 30 个项目可见），并按住 SHIFT 键点击中间的项目（第 25 个项目），则所选项目应该是从第 1 个到第 25 个，但前 20 个项目没有被渲染，因此我们无法通过已渲染的项目来检索已选择的项目。同样，**我们将使用故事中公开的变量**

这是故事的代码：

```js
export const WithSelectionManagement = () => {
  const items = getStoryItems({ amount: 10000 })

  const [selectedItems, setSelectedItems] = React.useState([])

  const handleSelect = React.useCallback(({ newSelectedIds }) => setSelectedItems(newSelectedIds), [
    setSelectedItems,
  ])

  // exposing data for Cypress
  React.useEffect(() => {
    global.storyData = {
      items,
      getItemText,
      selectedItems,
    }
  }, [items, selectedItems])

  return (
    <>
      <h4>The buttons must be clickable, the CTRL/CMD, ALT, SHIFT keyboard modifiers must work</h4>
      <VirtualList
        items={items}
        selectedItemIds={selectedItems}
        getItemHeights={() => 30}
        RenderItem={createSelectableRenderItem({ height: 30 })}
        listHeight={300}
        onSelect={handleSelect}
      />
    </>
  )
}
```

由于 VirtualList 组件是受控组件，它会将所选项目的列表——`selectedItems` 数组——传递给上层。父组件——即故事——向 Cypress 公开了该数组，以便进行所选项目的断言。
请注意：组件的创建，包括点击管理，已经移到了一个专用的高阶函数中：`createSelectableRenderdItem`。

选择单个项目的测试如下：

```js
it('When the items are clicked, then they are selected', () => {
  cy.visit('/iframe.html?id=virtuallist--with-selection-management')

  cy.window()
    .its('storyData')
    .should((storyData) => {
      // the story must expose some variables
      expect(storyData.items).to.be.to.have.length.of.at.least(3)
      expect(storyData.getItemText).to.be.a('function')
      expect(storyData.selectedItems).to.be.an('array')
    })
    .then(({ getItemText, items }) => {
      cy.findByText(getItemText(items[0]))
        .click()
        .window()
        .its('storyData.selectedItems')
        .should('eql', [items[0].id])

      cy.findByText(getItemText(items[1]))
        .click()
        .window()
        .its('storyData.selectedItems')
        .should('eql', [items[1].id])
    })
})
```

所有的断言都在公开的 `selectedItems` 数组本身上。我们不检查已渲染的项目是否被突出显示（突出显示项目的责任属于故事创建的组件），而是仅仅检查 VirtualList 组件是否正确处理了项目的点击。

`selectedItems` 数组不是在开始时读取的，而是在每次点击后读取，因为我们需要始终检查更新后的数组，而不是对初始数组的引用（`selectedItems` 由 `React.useState` 返回，因此在每次选择更新后都是新的）。

### 多项选择的断言

接下来的步骤是检查 VirtualList 组件是否正确处理了使用 Meta 键。在按住 Meta 键的情况下单击另一个项目应该导致两个项目被选中。

我们如何在 Cypress 中保持按住 Meta 键呢？只需：

```js
cy.get('body')
  // keeping pressed the meta (CMD) key
  .type('{meta}', { release: false })

// ...your test code...

cy.get('body')
  // releasing the meta (CMD) key
  .type('{meta}', { release: true })
```

添加项目点击将导致：

```js
cy.get('body')
  // keeping pressed the meta (CMD) key
  .type('{meta}', { release: false })
  .findByText(getItemText(items[2]))
  .click()
  .window()
  .its('storyData.selectedItems')
  .should('eql', [items[1].id, items[2].id])
  .get('body')
  // releasing the meta (CMD) key
  .type('{meta}', { release: true })

  cy.get('body')
    .type('{shift}', { release: false })
    .findByText(getItemText(firstRenderedItem))
    .click()
    .window()
    .its('storyData.selectedItems')
    .should('eql', expectedSelectedItemIds)
    .get('body')
    .type('{shift}', { release: true })
  })
```

这是测试结果的屏幕截图

![受控浏览器，左侧是所有断言结果，右侧是 Storybook 故事和选定的项目。](../../assets/images/cypress-storyboook/selection-test-result.png '测试的第一个结果：加载的故事和所有断言的结果。')_测试的第一个结果：加载的故事和所有断言的结果。_

### 滚动和选择

通过 `scrollVirtualList` 实用程序轻松滚动列表，通过 `getFirstRenderedItemIndex` 完成查找第一个已渲染项目，我们知道如何保持按键按下... 因此，测试滚动并带有 Shift 修饰键的点击（这意味着选择从先前点击的元素到最后一个元素的所有内容）应该只是计算所有（预计的）已选择项目的问题。下面的代码正是这样，继续查看完整的测试代码。

```js
cy.findAllByTestId('VirtualList')
  // scrolls the list
  .then(scrollVirtualList)
  .then(() => {
    // identifies the first rendered item (unknown in advance)
    const firstRenderedItemIndex = getFirstRenderedItemIndex(items, getItemText)
    const firstRenderedItem = items[firstRenderedItemIndex]

    // the tests is going to click on the first rendered item
    // keeping the SHIFT key pressed. All the items up to the first
    // rendered one should be selected
    const expectedSelectedItemIds = items
      .slice(0, firstRenderedItemIndex + 1)
      .map((item) => item.id)

    cy.get('body')
      .type('{shift}', { release: false })
      .findByText(getItemText(firstRenderedItem))
      .click()
      .window()
      .its('storyData.selectedItems')
      .should('eql', expectedSelectedItemIds)
      .get('body')
      .type('{shift}', { release: true })
  })
```

测试的完整代码，对每个键盘修饰键进行检查，可能会比较冗长但相当重复。

```js
it('When the items are clicked, then they are selected', () => {
  cy.visit('/iframe.html?id=virtuallist--with-selection-management')

  cy.window()
    .its('storyData')
    .should((storyData) => {
      // the story must expose some variables
      expect(storyData.items).to.be.to.have.length.of.at.least(3)
      expect(storyData.getItemText).to.be.a('function')
      expect(storyData.selectedItems).to.be.an('array')
    })
    .then(({ getItemText, items }) => {
      // first item click
      cy.findByText(getItemText(items[0]))
        .click()
        .window()
        .its('storyData.selectedItems')
        .should('eql', [items[0].id])

      // second item click
      cy.findByText(getItemText(items[1]))
        .click()
        .window()
        .its('storyData.selectedItems')
        .should('eql', [items[1].id])

      // third item click with Meta modifier
      cy.get('body')
        .type('{meta}', { release: false })
        .findByText(getItemText(items[2]))
        .click()
        .window()
        .its('storyData.selectedItems')
        .should('eql', [items[1].id, items[2].id])
        .get('body')
        .type('{meta}', { release: true })

      // first item click with Shift modifier
      cy.get('body')
        .type('{shift}', { release: false })
        .findByText(getItemText(items[0]))
        .click()
        .window()
        .its('storyData.selectedItems')
        .should('eql', [items[2].id, items[1].id, items[0].id])
        .get('body')
        .type('{shift}', { release: true })

      // second item click with Alt modifier
      cy.get('body')
        .type('{alt}', { release: false })
        .findByText(getItemText(items[1]))
        .click()
        .window()
        .its('storyData.selectedItems')
        .should('eql', [items[2].id, items[0].id])
        .get('body')
        .type('{alt}', { release: true })

      // scrolling
      cy.findAllByTestId('VirtualList')
        .then(scrollVirtualList)
        .then(() => {
          const firstRenderedItemIndex = getFirstRenderedItemIndex(items, getItemText)
          const firstRenderedItem = items[firstRenderedItemIndex]
          const expectedSelectedItemIds = items
            .slice(0, firstRenderedItemIndex + 1)
            .map((item) => item.id)

          // x-th item click with Shift modifier
          cy.get('body')
            .type('{shift}', { release: false })
            .findByText(getItemText(firstRenderedItem))
            .click()
            .window()
            .its('storyData.selectedItems')
            .should('eql', expectedSelectedItemIds)
            .get('body')
            .type('{shift}', { release: true })
        })
    })
})
```

[测试结果如下](https://www.youtube.com/watch?v=0DGHRJnWbHw)。

正如您所看到的，左侧的 Command Log 没有清楚地显示发生了什么。我们可以通过添加一些 `cy.log` 调用来提高测试的可读性，或者更好地利用 [Filip 的解决方案](https://medium.com/slido-dev-blog/cypress-tips-3-improve-your-error-screenshots-in-cypress-b3675968a190) 来充分利用 Cypress 的日志记录。

## 优化

测试运行得越快越好。我的原始测试套件花了将近 16 秒才能完全运行。[这里是视频录像](https://www.youtube.com/watch?v=EwBz844FuxE)（请注意，这是第一个测试套件，带有 60 FPS 测试）。

有两个主要的优化方向：

- 在故事之间更快地切换（而不是在测试之间重新加载页面）

- 通过 Cypress 强制浏览器时钟来加速滚动

### 在故事之间更快地切换

感谢 Nicholas Boll 和他的 [cypress-storybook](https://github.com/NicholasBoll/cypress-storybook) 插件，该插件利用 storybook APIs 在不重新加载整个页面的情况下加载所需的故事。

截至撰写本文时，我们只需要执行以下操作：

- 在 Storybook 配置中添加 `import 'cypress-storybook/react'`

- 在 Cypress 的 support/index.js 文件中添加 `import 'cypress-storybook/cypress'`

- 在测试文件中添加一个 `before` 钩子

```js
before(() => {
  // Visit the storybook iframe page once per file
  cy.visitStorybook()
})
```

- 通过 `cy.loadStory('VirtualList', 'With 10000 items')` 加载故事，而不是使用 `cy.visit`

- 在视频之后解释的公开变量上进行更新

这样做的话，页面在每个测试中都不会重新加载，整个测试套件节省了 3 秒，可以查看[这个视频](https://www.youtube.com/watch?v=UJhPTJhDEFM)。

我们为什么需要根据故事更新变量呢？嗯，如果页面没有重新加载，我们无法确保公开的变量是所需的变量，特别是因为几乎每个故事都会公开一个 `items` 变量，但变量的内容会有所不同。

我们如何区分故事公开的变量呢？嗯，公开故事名称就可以了！

```js
React.useEffect(() => {
  global.storyData = {
    // the name of the story is exposed too
    storyName: 'With 10000 items',
    items,
    visibleItemsAmount: Math.ceil(listHeight / itemHeight),
    getItemText,
  }
}, [items])
```

这样一来，每个测试都可以检查期望的名称。

```js
it('When the component receives 10000 items, then only the minimum number of items are rendered', () => {
  const story = 'With 10000 items'
  cy.loadStory('VirtualList', story)

  cy.window()
    .its('storyData')
    .should((storyData) => {
      // caring about the story name
      expect(storyData.storyName).to.eq(story)

      expect(storyData.items).to.be.to.have.length(10000)
      expect(storyData.visibleItemsAmount).to.be.greaterThan(0)
      expect(storyData.getItemText).to.be.a('function')
    })
    .then(({ visibleItemsAmount, getItemText, items }) => {
      // ... test code...
    })
})
```

利用 [Cypress 重试性](https://docs.cypress.io/guides/core-concepts/retry-ability.html) 来等待所有断言通过。这样我们确保公开的变量是正确的。

13 秒与最初的 16 秒相比可能听起来并不是巨大的改进… 但它是！因为如果我们在测试没有惯性滚动的组件（例如表单）的情况下，收益可能不是从 16 秒到 13 秒，而更有可能是从 9 秒到 6 秒！相信我，**永远不要低估测试速度**…

### 通过控制时钟加速滚动

视频突显了大部分测试持续时间都花在等待惯性滚动完成上。这并没有错，但一些测试运行器允许您控制时间流逝，Cypress 就是其中之一。在需要测试 `setTimeout` 或 `setInterval` 相关内容或惯性滚动之类的动画时，推动时间前进是至关重要的。

[官方文档](https://docs.cypress.io/api/commands/clock.html)中有很多示例，但对我们的需求来说，基本用法就足够了。我们需要调用 `cy.clock()`，然后使用 `cy.tick(<毫秒>)` 推动时间。

第一个使用新时钟控制的测试已注释。

```js
it('When the component is scrolled, then the rendered items are not the first ones', () => {
  const story = 'With 10000 items'
  cy.loadStory('VirtualList', story)

  // take control of the browser clock
  cy.clock()

  cy.window()
    .its('storyData')
    .should((storyData) => {
      expect(storyData.storyName).to.eq(story)
      expect(storyData.items).to.be.to.have.length(10000)
      expect(storyData.visibleItemsAmount).to.be.greaterThan(0)
      expect(storyData.getItemText).to.be.a('function')
    })
    .then(({ visibleItemsAmount, getItemText, items }) => {
      // this test does not need the `scrollVirtualList` utility anymore
      cy.findByTestId('VirtualList')
        .trigger('wheel', {
          deltaX: 0,
          deltaY: 1000,
        })

        // ticking the clock by one second.
        // It jumps to the inertial scroll end.
        .tick(1000)

        .then(() => getFirstRenderedItemIndex(items, getItemText))
        .should('be.greaterThan', 10)
        .then((firstVisibleItemIndex) => {
          const visibleItems = items.slice(
            firstVisibleItemIndex,
            firstVisibleItemIndex + visibleItemsAmount - 1
          )
          visibleItems.forEach((item) => cy.findByText(getItemText(item)).should('be.visible'))
        })
    })
})
```

[这是测试结果](https://www.youtube.com/watch?v=m6u0H8_jQuc)。

正如您所看到的，我们在惯性滚动结束时立即跳过。请注意：整个测试持续时间被屏幕录制所虚构，没有录制时测试所需的时间更短。

请记住，如果您使用了 `cy.clock`，使用 `cy.tick` 是强制的！如果您不手动进行时间推进，列表根本不会滚动，因为时钟被冻结了！看看如果不推进时钟会发生什么

![列表停留在初始位置和失败的测试。](../../assets/images/cypress-storyboook/clock-without-tick.png '_列表停留在初始滚动水平位置，因为我们没有推进时钟。')_列表停留在初始滚动水平位置，因为我们没有推进时钟._

`.should('be.greaterThan', 10)` 的断言未满足，因为列表根本没有滚动。

对几乎所有测试应用时钟控制，其持续时间减少了 9 秒。最终结果在下一个视频中

https://www.youtube.com/watch?v=Otf3J7qFAtI

最终结果：**从最初的 16 秒到 9 秒**，很棒！测试运行得越快，你就能更好地利用它们！

### “No preview”错误

如果遇到此错误

![Storybook 的“无预览”错误，显示在 Cypress 测试中。](../../assets/images/cypress-storyboook/no-preview-error.png)

重新启动 Cypress 应该足够了。否则，请重新启动 Storybook。这在几小时内仅发生了两次，所以我还没有深入研究它。

## 结论

简而言之，上述列出的解决方案是：

- 利用 [cypress-storybook](https://github.com/NicholasBoll/cypress-storybook) 允许 Cypress 快速切换故事

- 故事应该公开一些全局变量，允许 Cypress 进行数据断言

- 尽可能加速测试，例如在示例中，由于 Cypress 的时钟控制，惯性滚动不会等待

- 考虑是否要测试渲染的组件、故事代码或两者兼而有之

- 工具将更加集成，组件测试是目前的热门话题

<!-- TODO: 最后决定是否将所有资源移动到共同章节 -->

<br /><br />

_由 [NoriSte](https://github.com/NoriSte) 在 [dev.to](https://dev.to/noriste/testing-a-virtual-list-component-with-cypress-and-storybook-3lam) 和 [Medium](https://medium.com/@NoriSte/testing-a-virtual-list-component-with-cypress-and-storybook-494dc2d1d26b)上进行联合发表._
