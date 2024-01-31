# [过时] 使用 Cypress 进行 React 组件的单元测试

*此部分现已标记为过时，因为它指的是一个非常古老的 Cypress 版本（现在已完全支持组件测试）。*

---

_**更新**：[Cypress 10 发布，集成了组件测试和 E2E 测试](https://www.cypress.io/blog/2022/06/01/cypress-10-release/)，请查看并忽略下面报告的所有配置步骤，因为它们已经过时了！_

---

_**更新**：[Cypress 7 发布，支持全新的组件测试](https://docs.cypress.io/guides/component-testing/introduction#What-is-Component-Testing)，快去看看吧！而且，还有其他令人兴奋的消息即将到来，感谢[Storybook 6.2 发布](https://twitter.com/NoriSte/status/1378204109841571840)！_

---

如今，使用 Cypress 进行 React 组件的单元测试已经成为可能，这是 [使用 Cypress 和 Storybook 进行组件测试](./cypress-and-storybook.zh.md) 的一个扩展章节。

前一章的目标是在**React 组件测试世界**中进行一些实验，这是现今一个非常重要的主题。

动机非常简单：

- 你可能已经在你的团队中使用了 [Storybook](https://storybook.js.org/)（如果没有，请考虑添加！）

- 你可能不熟悉使用 [Testing Library](https://testing-library.com/) 进行组件测试，或者你对 JSDom 有一些偏见，或者你希望在真实浏览器中测试你的 UI 组件，而不是在模拟的 DOM 环境中进行测试。

- 你可能已经熟悉 [Cypress](https://www.cypress.io/) 或 [TestCafé](https://devexpress.github.io/testcafe/)（如果没有，请考虑它们用于你的 UI 测试），并且你可能希望只使用一个工具进行测试。

方法也很简单：

- 将故事的 props 暴露给测试工具，用于控制渲染的组件。

- 从 Cypress/TestCafé 中获取它们，自动执行用户操作并断言 props 的内容。

但是也有**一些注意事项**…

- 性能问题：在[章节](https://github.com/NoriSte/all-my-contributions)中，我花费了额外的努力来最小化故事切换导致的速度下降。

- **测试和故事的耦合性**：由于 Cypress 甚至都使用了 Storybook，故事不仅要为团队共享的设计系统负责，还要为组件测试负责。

- **回调测试变得困难**：检查回调 props 的参数和调用变得困难。

我的实验中的一些问题可以通过 [daedalius 的方法](./cypress-and-storybook-exposing-component-from-story.zh.md) 缓解，但解决方案还不够理想，但是然后…

## Cypress 4.5.0 已发布

在 4 月 28 日，Cypress 4.5.0 版本发布了，唯一的功能更新如下

> 当将 experimentalComponentTesting 配置选项设置为 true 时，Cypress 现在支持使用特定于框架的适配器执行组件测试。有关详细信息，请参见 [cypress-react-unit-test](https://github.com/bahmutov/cypress-react-unit-test) 和 [cypress-vue-unit-test](https://github.com/bahmutov/cypress-vue-unit-test) 仓库。

这是什么意思呢？这意味着 Cypress 现在可以直接挂载 React 组件，为 **[cypress-react-unit-test](https://github.com/bahmutov/cypress-react-unit-test)** 带来了新的生机！在 Cypress 4.5.0 发布之前，该插件相当有限，但现在它得到了一流的支持！事实上，cypress-react-unit-test 现在非常可靠，是一个有意义的插件。

## 测试 VirtualList 组件：第二集

组件依然是 VirtualList，关于它的更多信息请查看[上一章](./cypress-and-storybook.zh.md)。我们需要同时设置 [cypress-react-unit-test](https://github.com/bahmutov/cypress-react-unit-test) 和 TypeScript 转换（该组件使用 TypeScript 编写，是 [Lerna](https://github.com/lerna/lerna) monorepo 的一部分，并且使用 Webpack 进行编译）。这两个步骤都很简单，但是由于插件在其文档中有一个[专门的安装部分](https://github.com/bahmutov/cypress-react-unit-test#install)，因此 TypeScript 编译可能不太明显，而且有许多过时或部分过时的不同方法和资源。

最简洁而有效的解决方案是[André Pena 的方法](https://stackoverflow.com/a/60017105/700707)，因此我只需要：

- 添加 _cypress/webpack.config.js_ 文件

```js
module.exports = {
  mode: 'development',
  devtool: false,
  resolve: {
    extensions: ['.ts', '.tsx', '.js'],
  },
  module: {
    rules: [
      {
        test: /\.tsx?$/,
        exclude: [/node_modules/],
        use: [
          {
            loader: 'ts-loader',
            options: {
              // skip typechecking for speed
              transpileOnly: true,
            },
          },
        ],
      },
    ],
  },
}
```

- 添加  _cypress/tsconfig.json_ 文件

```json
{
  "extends": "../tsconfig.json",
  "compilerOptions": {
    "types": ["cypress", "cypress-wait-until"]
  }
}
```

请注意：

- ../tsconfig.json 文件与 React 应用程序使用相同的文件

- [cypress-wait-until](https://github.com/NoriSte/cypress-wait-until) 不是必需的，但我经常使用它，它是 Cypress 最常安装的插件之一

上述与转译相关的文件，以及以下的 cypress.json 文件

```json
{
  "experimentalComponentTesting": true,
  "componentFolder": "cypress/component"
}
```

以上就足够开始尝试编写 cypress/component/VirtualList.spec.tsx 测试了！在前一章中，第一个测试是标准渲染，即“当组件接收到 10000 个项时，只渲染最少数量的项”测试，就这样吧：

```typescript
/// <reference types="Cypress" />
/// <reference types="cypress-wait-until" />

import React from 'react'
import { mount } from 'cypress-react-unit-test'
import '[@testing](http://twitter.com/testing)-library/cypress/add-commands'

import { VirtualList } from '../../src/atoms/VirtualList'
import { getStoryItems } from '../../stories/atoms/VirtualList/utils'

describe('VirtualList', () => {
  it('When the list receives 10000 items, then only the minimum number of them are rendered', () => {
    // Arrange
    const itemsAmount = 10000
    const itemHeight = 30
    const listHeight = 300
    const items = getStoryItems({ amount: itemsAmount })
    const visibleItemsAmount = listHeight / itemHeight

    // Act
    mount(
      <VirtualList
        items={items}
        getItemHeights={() => itemHeight}
        RenderItem={createRenderItem({ height: itemHeight })}
        listHeight={listHeight}
      />
    )

    // Assert
    const visibleItems = items.slice(0, visibleItemsAmount - 1)
    itemsShouldBeVisible(visibleItems)

    // first not-rendered item check
    cy.findByText(getItemText(items[visibleItemsAmount])).should('not.exist')
  })
})
```

与 Storybook 相关的章节相比：

- 这

```js
/// <reference types="Cypress" />
/// <reference types="cypress-wait-until" />
```

在开始时需要这些内容，以便让 VSCode 正确利用 TypeScript 的建议和错误报告

- 我们使用 cypress-react-unit-test 的 mount API 来挂载组件，如果你习惯于 [Testing Library APIs](https://testing-library.com/docs/react-testing-library/api)，这并没有什么特别的新东西

没有更多，Cypress 测试与 Storybook 相关的测试一样 😊

### 回调测试

将所有测试从[上一章](./cypress-and-storybook.zh.md)迁移过来相当容易，缺失的是“选择测试”的回调测试部分。

创建一个名为 _WithSelectionManagement_ 的包装组件，它渲染 _VirtualList_ 并管理项的选择是相当容易的，我们可以将我们的存根传递给它并对其进行断言。

```typescript
it('When the items are clicked, then they are selected', () => {
  const itemHeight = 30
  const listHeight = 300
  let testItems

  const WithSelectionManagement: React.FC<{
    testHandleSelect: (newSelectedIds: ItemId[]) => {}
  }> = (props) => {
    const { testHandleSelect } = props
    const items = getStoryItems({ amount: 10000 })

    const [selectedItems, setSelectedItems] = React.useState<(string | number)[]>([])

    const handleSelect = React.useCallback<(params: OnSelectCallbackParams<StoryItem>) => void>(
      ({ newSelectedIds }) => {
        setSelectedItems(newSelectedIds)
        testHandleSelect(newSelectedIds)
      },
      [setSelectedItems, testHandleSelect]
    )

    React.useEffect(() => {
      testItems = items
    }, [items])

    return (
      <VirtualList
        items={items}
        getItemHeights={() => itemHeight}
        listHeight={listHeight}
        RenderItem={createSelectableRenderItem({ height: itemHeight })}
        selectedItemIds={selectedItems}
        onSelect={handleSelect}
      />
    )
  }
  WithSelectionManagement.displayName = 'WithSelectionManagement'

  mount(<WithSelectionManagement testHandleSelect={cy.stub().as('handleSelect')} />)

  cy.then(() => expect(testItems).to.have.length.greaterThan(0))
  cy.wrap(testItems).then(() => {
    cy.findByText(getItemText(testItems[0])).click()
    cy.get('[@handleSelect](http://twitter.com/handleSelect)').should((stub) => {
      expect(stub).to.have.been.calledOnce
      expect(stub).to.have.been.calledWith([testItems[0].id])
    })
  })
})
```

请参考完整的 SinonJS（由 Cypress 包装和使用）[Stub](https://sinonjs.org/releases/v9.0.2/stubs/)/[Spy](https://sinonjs.org/releases/v9.0.2/spies/) 文档，了解详细的 API。

## 总结

请查看[此视频](https://youtu.be/yVL-yRT5hFQ)中的结果。现在测试只需要不到七秒，无需依赖或加载 Storybook，充分利用 Cypress 的一流支持。

接下来呢？[cypress-react-unit-test](https://github.com/bahmutov/cypress-react-unit-test) 插件目前相当稳定和实用，一个全新的实验领域正向我们展开，许多中小型项目可以选择将 Cypress 作为唯一的测试工具。

<!-- TODO: 最后决定是否要将所有资源移到一个共同的章节中 -->

<br /><br />

_由 [NoriSte](https://github.com/NoriSte) 在 [dev.to](https://dev.to/noriste/unit-testing-react-components-with-cypress-5fk2) 和 [Medium](https://medium.com/@NoriSte/unit-testing-react-components-with-cypress-4d4cf8cd59a0)上进行联合发表._
