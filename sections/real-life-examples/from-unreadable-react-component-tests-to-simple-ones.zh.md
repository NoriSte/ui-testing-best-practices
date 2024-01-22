# 从难以理解的 React 组件测试到简单愚蠢的测试

<br/><br/>

## 一段简要说明

测试的代码应该尽量清晰易懂。这样做的好处是在需要理解、更新、重构或修复代码时，能够节省大量时间。相反，如果你连自己写的测试都读不懂，那可就是个糟糕的场景了！

<br/><br/>

这里我将分享一下我对一些旧的 React 组件测试进行重构的原因、思考过程和模式。

## 问题

在添加了["使用 Cypress 和 Storybook 测试虚拟列表组件"](../../sections/tools/cypress-and-storybook-exposing-component-from-story.zh.md)章节和["使用 Cypress 进行 React 组件单元测试"](../../sections/tools/cypress-react-component-test.zh.md)章节一年后，我发现我的测试几乎无法阅读。许多抽象层次使我无法仔细理解测试在做什么，导致花费很长时间阅读它们。这是无法接受的阻力。

### 如何提高测试的可读性？

TDD 之父 Kent Beck 曾说：

> 测试代码不是生产代码。它必须简单且小 1000 倍。

但是，如何做到呢？是什么导致我的测试如此难以阅读？是什么使可读性变差？

我将分析一堆我的测试，对它们进行思考，并提出更为直观的方法。被测试的组件是前文提到的 VirtualList。

#### 测试 1，复杂度低

接下来的测试是最简单的一个：检查当没有传递任何内容时，VirtualList 是否不渲染任何东西。以下是原始测试

```tsx
it('When there are no items, then nothing is showed', () => {
  const itemsAmount = 0
  const itemHeight = 30
  const listHeight = 300
  const items = getStoryItems({ amount: itemsAmount })

  mount(
    <VirtualList
      items={items}
      getItemHeights={() => itemHeight}
      RenderItem={createRenderItem({ height: itemHeight })}
      listHeight={listHeight}
    />,
  )

  cy.findByTestId('VirtualList')
    .then($el => $el.text())
    .should('be.empty')
})
```

考虑到测试的简单性，谈论可读性是否有点过于琐碎？不，我已经对此有一些疑问，比如：

- `getStoryItems` 是做什么的？
- `createRenderItem` 是做什么的？

我这里没有贴出 `getStoryItems` 的代码，但它只是一个生成包含 X 个项目的数组的函数，形式为 `[{ id: 1, name: 'Item 1' }, { id: 2, name: 'Item 2' }, /* 等等 */]`。它的目的是轻松生成数千个项目，但我编写它的初衷是为了更轻松地编写没有测试意识的 Storybook 故事！然后，我将其重新用于测试，但故事和测试有完全不同的需求和目的！

与此同时，`createRenderItem` 是一个工厂，生成列表项（一些 React 组件）。同样，我为 Storybook 设计它是为了使其更具动态性，而测试并不是我最初考虑的。

这两个函数都很容易阅读，但为什么要强迫读者跟随这些抽象？它们是必需的吗？答案是否定的。

以下是测试的简化代码，列出了其中的差异。

```tsx
it('When there are no items, then nothing is showed', () => {
  // ------------------------------------------
  // Arrange
  const items = [];

  mount(
    <VirtualList
      items={items}
      listHeight={90}
      getItemHeights={() => 30}
      RenderItem={RenderItem}
    />
  );

  // ------------------------------------------
  // Assert
  cy.findByTestId('VirtualList')
    .then(($el) => $el.text())
    .should('be.empty');
});
```

```diff
-const itemsAmount = 0
-const itemHeight = 30
-const listHeight = 300
-const items = getStoryItems({ amount: itemsAmount })
+const items = [];

mount(
  <VirtualList
    items={items}
    listHeight={listHeight}
-   getItemHeights={() => itemHeight}
+   getItemHeights={() => 30}
-   RenderItem={createRenderItem({ height: itemHeight })}
+   RenderItem={RenderItem}
  />,
)
```

在简化的测试中，最显著的变化有：

1. 我显式指定了 `items`，而不是通过 `getStoryItems` 生成它们。这样做的目标是立即向读者澄清 VirtualList 渲染的是哪些项目。
2. 我移除了“不必要的”常量。如果没有渲染任何项目，项目的高度和列表的高度是无关紧要的。
3. 我取消了 `createRenderItem` 工厂的必要性。在这里，生成预先设置高度的组件是无用的。

#### 测试 2，中等复杂度

接下来的测试并不复杂。但我实现它的方式在不必要的情况下增加了复杂度

```tsx
it('When the list receives 10000 items, then only the minimum number of them are rendered', () => {
  const itemsAmount = 10000
  const itemHeight = 30
  const listHeight = 300
  const items = getStoryItems({ amount: itemsAmount })
  const visibleItemsAmount = listHeight / itemHeight

  mount(
    <VirtualList
      items={items}
      getItemHeights={() => itemHeight}
      RenderItem={createRenderItem({ height: itemHeight })}
      listHeight={listHeight}
    />,
  )

  const visibleItems = items.slice(0, visibleItemsAmount - 1)
  itemsShouldBeVisible(visibleItems)

  // first not-rendered item check
  cy.findByText(getItemText(items[visibleItemsAmount])).should('not.exist')
})
```

在这里情况变得更加奇怪。再次出现了一些简单的抽象，但读者必须：

- 理解 `visibleItemsAmount` 的值。
- 通过阅读 `items.slice` 理解 `visibleItems` 包含什么。再一次，多了一层复杂性。
- 猜测 `itemsShouldBeVisible` 是做什么的，或者阅读它的代码。
- 猜测 `items[visibleItemsAmount]` 包含什么。

是的，有一些注释。是的，我尝试创建有意义的名称。但我可以简化它很多。

还有一件事。想象一下这种情况：测试失败了。不管你做了什么，这个测试都失败了。你开始调试它，但你知道调试一个高度动态的测试（你不知道 `items`，`visibleItemsAmount`，`visibleItems`，`items[visibleItemsAmount]` 在前期包含什么）是很困难的。你需要在测试代码中添加 `console.log`/`cy.log`/`debugger`/断点，以对测试管理的内容有一个整体的概念，然后才能开始调试 VirtualList。我能避免未来的调试问题吗？我需要测试渲染 10,000 个项目吗？

以下是我如何改进测试的方式。

```tsx
it('When the list is longer than the available space, then only the minimum number of items are rendered', () => {
  // ------------------------------------------
  // Arrange

  // creating the data
  const items = [
    // visible ones
    { id: 1, name: 'Item 1' },
    { id: 2, name: 'Item 2' },
    { id: 3, name: 'Item 3' },
    // invisible one
    { id: 3, name: 'Item 4' },
  ];

  // only 3 items are visible
  const itemHeight = 30;
  const listHeight = 90;

  // mounting the component
  mount(
    <VirtualList
      items={items}
      getItemHeights={() => itemHeight}
      RenderItem={RenderItem}
      listHeight={listHeight}
    />
  );

  // ------------------------------------------
  // Act

  // ------------------------------------------
  // Assert
  cy.findByText('Item 1').should('be.visible');
  cy.findByText('Item 2').should('be.visible');
  cy.findByText('Item 3').should('be.visible');
  cy.findByText('Item 4').should('not.exist');
});
```

同样，在这里，唯一而且关键的改变是为未来的读者/调试者提供了更轻松的体验。你不需要猜测/计算/记录常量的值，也不需要弄清楚抽象的作用。代码就在你眼前，遵循 KISS 原则（保持简单，傻瓜）。

```diff
-const itemsAmount = 10000
-const itemHeight = 30
-const listHeight = 300
-const items = getStoryItems({ amount: itemsAmount })
-const visibleItemsAmount = listHeight / itemHeight
+// creating the data
+const items = [
+ // visible ones
+ { id: 1, name: 'Item 1' },
+ { id: 2, name: 'Item 2' },
+ { id: 3, name: 'Item 3' },
+ // invisible one
+ { id: 3, name: 'Item 4' },
+];

+// only 3 items are visible
+const itemHeight = 30;
+const listHeight = 90;

/* ... */

-const visibleItems = items.slice(0, visibleItemsAmount - 1)
-itemsShouldBeVisible(visibleItems)
-cy.findByText(getItemText(items[visibleItemsAmount])).should('not.exist')
+cy.findByText('Item 1').should('be.visible');
+cy.findByText('Item 2').should('be.visible');
+cy.findByText('Item 3').should('be.visible');
+cy.findByText('Item 4').should('not.exist');
```

#### 测试 3，中等复杂度

在下一个测试中，我们的目标是测试 VirtualList 的 `buffer` 属性，该属性允许在项目尚不可见时渲染一些项目。

```tsx
it('When some items buffered, then they exist in the page', () => {
  const itemsAmount = 1000
  const itemHeight = 30
  const listHeight = 300
  const items = getStoryItems({ amount: itemsAmount })
  const visibleItemsAmount = listHeight / itemHeight
  const bufferedItemsAmount = 5

  mount(
    <VirtualList
      items={items}
      getItemHeights={() => itemHeight}
      RenderItem={createRenderItem({ height: itemHeight })}
      listHeight={listHeight}
      buffer={bufferedItemsAmount}
    />,
  )

  fastScrollVirtualList().then(() => {
    const firstRenderedItemIndex = getFirstRenderedItemIndex(items, getItemText)
    const firstVisibleItemIndex = firstRenderedItemIndex + bufferedItemsAmount
    const lastVisibleItemIndex = firstVisibleItemIndex + visibleItemsAmount + 1
    const lastRenderedItemIndex = lastVisibleItemIndex + bufferedItemsAmount

    items.slice(firstRenderedItemIndex, firstVisibleItemIndex).forEach(item => {
      cy.findByText(getItemText(item)).should('not.be.visible')
    })

    items.slice(firstVisibleItemIndex, lastVisibleItemIndex).forEach(item => {
      cy.findByText(getItemText(item)).should('be.visible')
    })

    items.slice(lastVisibleItemIndex, lastRenderedItemIndex).forEach(item => {
      cy.findByText(getItemText(item)).should('not.be.visible')
    })
  })
})
```

复杂性来自于：

1. 滚动列表以测试缓冲区在两侧的作用。
2. 当滚动停止时，我无法提前知道哪些项目是可见的，哪些不可见，这就是为什么 `fastScrollVirtualList().then(() => { /* ... */ })` 的内容是动态的。

我稍后会回到第 2 点，但对于这个测试，我切掉了第 1 点的头：为什么我需要通过 Cypress 组件测试在这里测试整个 `buffer` 行为？我有很多单元测试，检查内部 VirtualList 函数是否正常工作。我不需要再次测试相同的行为。一旦 `buffer` 生效，它就在两侧都生效了。这个选择使测试受益匪浅；现在它更简单了，包含了许多有助于读者的注释！

```tsx
it('Should render only the visible items and the buffered ones when an item is partially visible', () => {
  // ------------------------------------------
  // Arrange

  // creating the data
  const items = [
    // visible ones
    { id: 1, name: 'Item 1' },
    { id: 2, name: 'Item 2' },
    { id: 3, name: 'Item 3' },
    // visible ones
    { id: 4, name: 'Item 4' },
    // buffered ones
    { id: 5, name: 'Item 5' },
    { id: 6, name: 'Item 6' },
    // non-rendered one
    { id: 7, name: 'Item 7' },
  ];

  const itemHeight = 30;
  // 3 items are fully visible, 1 is partially visible
  const listHeight = 100;
  // 2 items are buffered
  const buffer = 2;

  // mounting the component
  mount(
    <VirtualList
      items={items}
      getItemHeights={() => itemHeight}
      RenderItem={RenderItem}
      listHeight={listHeight}
      buffer={buffer}
    />
  );

  // ------------------------------------------
  // Act

  // ------------------------------------------
  // Assert
  cy.findByText('Item 1').should('be.visible');
  cy.findByText('Item 2').should('be.visible');
  cy.findByText('Item 3').should('be.visible');
  cy.findByText('Item 4').should('be.visible');
  cy.findByText('Item 5').should('not.be.visible');
  cy.findByText('Item 6').should('not.be.visible');
  cy.findByText('Item 7').should('not.exist');
});
```

#### 测试 4，高复杂度

测试选择是 VirtualList 中最困难的部分，因为：

1. VirtualList 管理键盘修饰符，允许简单、累加、减去和范围选择。
2. VirtualList 是无状态的：我们需要用一个有状态的包装器来包装它，以存储先前的选择，以便测试累加、减去和范围选择。

我的旧测试非常难以阅读，因为：

1. 有状态的包装器在测试本身的主体中声明，使用测试的范围。
2. 所有的选择都在一个很长的流中被检查。

逐步来，让我们简化它。

##### 将抽象移到远处

旧测试的第一部分如下：

```tsx
it('When the items are clicked, then they are selected', () => {
  const itemHeight = 30
  const listHeight = 300
  let testItems

  const WithSelectionManagement: React.FC<{
    testHandleSelect: (newSelectedIds: ItemId[]) => {}
  }> = props => {
    const { testHandleSelect } = props
    const items = getStoryItems({ amount: 10000 })

    const [selectedItems, setSelectedItems] = React.useState<(string | number)[]>([])

    const handleSelect = React.useCallback<(params: OnSelectCallbackParams<StoryItem>) => void>(
      ({ newSelectedIds }) => {
        setSelectedItems(newSelectedIds)
        testHandleSelect(newSelectedIds)
      },
      [setSelectedItems, testHandleSelect],
    )

    React.useEffect(() => {
      testItems = items
    }, [items])

    return (
      <VirtualList
        items={items}
        getItemHeights={() => itemHeight}
        RenderItem={createSelectableRenderItem({ height: itemHeight })}
        listHeight={listHeight}
        updateScrollModeOnDataChange={{
          addedAtTop: true,
          removedFromTop: true,
          addedAtBottom: true,
          removedFromBottom: true,
        }}
        selectedItemIds={selectedItems}
        onSelect={handleSelect}
      />
    )
  }
  WithSelectionManagement.displayName = 'WithSelectionManagement'

  mount(<WithSelectionManagement testHandleSelect={cy.stub().as('handleSelect')} />)

  /* ... rest of the test ... */
})
```

以这样的样板开始阅读测试是相当困难的。你在阅读中迷失了一段时间，失去了测试本身的关键部分。让我们将其从测试中移到远处。

```tsx
// wrap the VirtualList to internally manage the selection, passing outside only the new selection
function SelectableList(props) {
  const { onSelect, ...virtualListProps } = props;

  // store the selection in an internal state
  const [selectedItems, setSelectedItems] = React.useState([]);
  const handleSelect = React.useCallback(
    ({ newSelectedIds }) => {
      setSelectedItems(newSelectedIds);
      // call the passed spy to notify the test about the new selected ids
      onSelect({ newSelectedIds });
    },
    [setSelectedItems, onSelect]
  );

  // Transparently renders the VirtualList, apart from:
  // - storing the selection
  // - passing the new selection back to the test
  return (
    <VirtualList
      selectedItemIds={selectedItems}
      onSelect={handleSelect}
      // VirtualList props passed from the test
      {...virtualListProps}
    />
  );
}

it('When two items are clicked pressing the meta button, then they are both selected', () => {
    // ------------------------------------------
    // Arrange

    // creating the data
    const itemHeight = 30;
    const listHeight = 90;
    const items = [
      { id: 1, name: 'Item 1' },
      { id: 2, name: 'Item 2' },
      { id: 3, name: 'Item 3' },
    ];

    // mounting the component
    mount(
      <SelectableList
        // test-specific props
        onSelect={cy.spy().as('onSelect')}
        // VirtualList props
        items={items}
        getItemHeights={() => itemHeight}
        RenderItem={RenderItem}
        listHeight={listHeight}
      />
    );

  /* ... rest of the test ... */
})
```

背景干扰仍然很高，但在阅读测试时你不会遇到它。然后，你是否注意到包装器的调用消耗从

```tsx
mount(<WithSelectionManagement testHandleSelect={cy.stub().as('handleSelect')} />)
```

到

```tsx
mount(
  <SelectableList
    // test-specific props
    onSelect={cy.spy().as('onSelect')}
    // VirtualList props
    items={items}
    getItemHeights={() => itemHeight}
    RenderItem={RenderItem}
    listHeight={listHeight}
  />
);
```

？目的是尽可能使代码与之前的更简单的测试相似。`SelectableList` 现在更透明，因为它只做了一件事情：存储先前的选择。

##### 拆解长的测试

系好安全带：除了包装器，旧测试如下：

```tsx
it('When the items are clicked, then they are selected', () => {
  /* ... the code of the wrapper ... */

  cy.then(() => expect(testItems).to.have.length.greaterThan(0))
  cy.wrap(testItems).then(() => {
    cy.findByText(getItemText(testItems[0])).click()
    cy.get('@handleSelect').should(stub => {
      expect(stub).to.have.been.calledOnce
      expect(stub).to.have.been.calledWith([testItems[0].id])
    })

    cy.findByText(getItemText(testItems[1])).click().window()
    cy.get('@handleSelect').should(stub => {
      expect(stub).to.have.been.calledTwice
      expect(stub).to.have.been.calledWith([testItems[1].id])
    })

    cy.get('body')
      .type('{meta}', { release: false })
      .findByText(getItemText(testItems[2]))
      .click()
      .get('@handleSelect')
      .should(stub => {
        expect(stub).to.have.been.calledThrice
        expect(stub).to.have.been.calledWith([testItems[1].id, testItems[2].id])
      })
      .get('body')
      .type('{meta}', { release: true })

    cy.get('body')
      .type('{shift}', { release: false })
      .findByText(getItemText(testItems[0]))
      .click()
      .get('@handleSelect')
      .should(stub => {
        expect(stub).to.have.been.callCount(4)
        expect(stub).to.have.been.calledWith([testItems[2].id, testItems[1].id, testItems[0].id])
      })
      .get('body')
      .type('{shift}', { release: true })

    cy.get('body')
      .type('{alt}', { release: false })
      .findByText(getItemText(testItems[1]))
      .click()
      .get('@handleSelect')
      .should(stub => {
        expect(stub).to.have.been.callCount(5)
        expect(stub).to.have.been.calledWith([testItems[2].id, testItems[0].id])
      })
      .get('body')
      .type('{alt}', { release: true })

    fastScrollVirtualList().then(() => {
      const firstRenderedItemIndex = getFirstRenderedItemIndex(testItems, getItemText)
      const firstRenderedItem = testItems[firstRenderedItemIndex]
      const expectedSelectedItemIds = testItems
        .slice(0, firstRenderedItemIndex + 1)
        .map(item => item.id)

      cy.get('body')
        .type('{shift}', { release: false })
        .findByText(getItemText(firstRenderedItem))
        .click()
        .get('@handleSelect')
        .should(stub => {
          expect(stub).to.have.been.callCount(6)
          expect(stub).to.have.been.calledWith(expectedSelectedItemIds)
        })
        .get('body')
        .type('{shift}', { release: true })
    })
  })
})
```

现在看来，看起来有点奇怪的地方：

- `cy.then(() => expect(testItems).to.have.length.greaterThan(0))`，因为项目是动态生成的，我在外部函数出现问题时中止了测试。
- `cy.findByText(getItemText(testItems[0])).click()` 太过动态，`testItems[0]` 包含什么？
- `fastScrollVirtualList().then(() => { /* ... */ }` 的内容 😩
- 测试本身的长度。
- 不清楚测试一种选择类型在哪里结束，下一个选择类型在哪里开始。

让我们首先通过将一个包含四种选择的测试拆分为四个测试来消除对这样一个长测试的需求。

###### 单选

猜猜看？测试简单的选择根本不需要包装器 😊

```tsx
it('When an item is clicked, then it is selected', () => {
  // ------------------------------------------
  // Arrange

  // creating the spy
  // a spy is needed to intercept the call the VirtualList does
  const onSelectSpy = cy.spy().as('onSelect');

  // creating the data
  const items = [
    { id: 1, name: 'Item 1' },
    { id: 2, name: 'Item 2' },
    { id: 3, name: 'Item 3' },
  ];

  // mounting the component
  mount(
    <VirtualList
      items={items}
      getItemHeights={() => 30}
      RenderItem={RenderItem}
      listHeight={90}
      onSelect={onSelectSpy}
    />
  );

  // ------------------------------------------
  // Act
  cy.findByText('Item 1').click();

  // ------------------------------------------
  // Assert
  cy.get('@onSelect').should((spy) => {
    expect(spy).to.have.been.calledOnce;

    // Sinon matchers allow to assert about partials of the params
    // see
    // https://sinonjs.org/releases/latest/assertions/
    // https://sinonjs.org/releases/latest/matchers/
    expect(spy).to.have.been.calledWith(
      Cypress.sinon.match({ newSelectedIds: [1] })
    );
    expect(spy).to.have.been.calledWith(
      Cypress.sinon.match({ item: { id: 1, name: 'Item 1' } })
    );
  });
});
```

与旧测试的第一部分相比，最显著的变化：

- 不再需要包装器。
- 不再有动态值。
- 更有表现力的断言。
- 这是一个单一目的的测试。

###### 累加和减去选择

在这里，我们需要利用包装器。

```tsx
it('When two items are clicked pressing the meta button, then they are both selected', () => {
    // ------------------------------------------
    // Arrange

    // creating the data
    const itemHeight = 30;
    const listHeight = 90;
    const items = [
      { id: 1, name: 'Item 1' },
      { id: 2, name: 'Item 2' },
      { id: 3, name: 'Item 3' },
    ];

    // mounting the component
    mount(
      // mount `SelectableList`instead of `VirtualList`
      <SelectableList
        // test-specific props
        onSelect={cy.spy().as('onSelect')}
        // VirtualList props
        items={items}
        getItemHeights={() => itemHeight}
        RenderItem={RenderItem}
        listHeight={listHeight}
      />
    );

    // ------------------------------------------
    // click on the first item
    // Act
    cy.findByText('Item 1')
      .click()
      // Assert
      .get('@onSelect')
      .should((spy) => {
        expect(spy).to.have.been.calledOnce;
        expect(spy).to.have.been.calledWith({ newSelectedIds: [1] });
      });

    // ------------------------------------------
    // click on the second item
    // Act
    // keep the meta button  pressed
    cy.get('body').type('{meta}', { release: false });

    cy.findByText('Item 2')
      .click()
      // Assert
      .get('@onSelect')
      .should((spy) => {
        expect(spy).to.have.been.calledTwice;
        expect(spy).to.have.been.calledWith({ newSelectedIds: [1, 2] });
      });
  });
```

这里的巨大差异在于避免滚动，因为它不影响累加选择。再次出现显式值，清晰度和简洁性允许进行更直观和更有表现力的测试。

测试减去选择遵循相同的模式。我在这里不报告它。

###### 范围选择

与之前的选择相比，我认为范围选择 **可能会** 受到滚动列表的影响，因为一些将被选择的项目不再呈现。如何使本质上是动态的东西——滚动列表——更加静态？通过手动测试和一些注释。

```tsx
it('When the list is scrolled amd some items are clicked pressing the shift button, then the range selection works', () => {
  // ------------------------------------------
  // Arrange

  // creating the data
  const items = [
    { id: 1, name: 'Item 1' },
    { id: 2, name: 'Item 2' },
    { id: 3, name: 'Item 3' },
    { id: 4, name: 'Item 4' },
    { id: 5, name: 'Item 5' },
    { id: 6, name: 'Item 6' },
    { id: 7, name: 'Item 7' },
    { id: 8, name: 'Item 8' },
    { id: 9, name: 'Item 9' },
    { id: 10, name: 'Item 10' },
    { id: 11, name: 'Item 11' },
    { id: 12, name: 'Item 12' },
    { id: 13, name: 'Item 13' },
    { id: 14, name: 'Item 14' },
    { id: 15, name: 'Item 15' },
    { id: 16, name: 'Item 16' },
    { id: 17, name: 'Item 17' },
    { id: 18, name: 'Item 18' },
    { id: 19, name: 'Item 19' },
    { id: 20, name: 'Item 20' },
  ];
  // only 3 items are visible
  const itemHeight = 30;
  const listHeight = 90;

  // mounting the component
  mount(
    <SelectableList
      // test-specific props
      onSelect={cy.spy().as('onSelect')}
      // VirtualList props
      items={items}
      getItemHeights={() => itemHeight}
      RenderItem={RenderItem}
      listHeight={listHeight}
    />
  );

  // Act

  // click on the first item
  cy.findByText('Item 1').click();

  // scroll the list
  cy.findAllByTestId('VirtualList').first().trigger('wheel', {
    deltaX: 0,
    deltaY: 200,
  });

  cy.get('body').type('{shift}', { release: false });

  // Item 7, 8, 9, and 10 are going to be visible after the scroll
  // click on the 10th item. It's going to be clicked as soon as it's in the DOM and clickable
  cy.findByText('Item 10')
    .click()
    // Assert
    .get('@onSelect')
    .should((spy) => {
      expect(spy).to.have.been.calledTwice;
      expect(spy).to.have.been.calledWith({ newSelectedIds: [1, 2, 3, 4, 5, 6, 7, 8, 9, 10] });
    });
});
```

这里可能发生的最糟糕的事情是，在触发 `wheel` 事件时，VirtualList 的滚动逻辑滚动得更少或更多。

```tsx
cy.findAllByTestId('VirtualList').first().trigger('wheel', {
  deltaX: 0,
  deltaY: 200,
});
```

但如果发生了，由于有注释，读者会清楚地知道出了什么问题 😊

## 相关的文章

- 🔗 [保持低抽象度以便于调试测试](/sections/generic-best-practices/test-code-with-debugging-in-mind.zh.md)

<br/><br/>

*由[NoriSte](https://github.com/NoriSte)在[dev.to](https://dev.to/noriste/from-unreadable-react-component-tests-to-simple-stupid-ones-3ge6)进行联合发表。*
