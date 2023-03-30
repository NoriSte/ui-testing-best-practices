# From unreadable React Component Tests to simple, stupid ones

<br/><br/>

### One Paragraph Explainer

The test's code must be as straightforward as possible. The benefit is to save a lot of time to understand, update, refactor, fix it when needed. At the opposite, a terrible scenario happens when you are not able to read some tests, even if you are the author!

<br/><br/>

Here are the rationales, the mental process, and the patterns at the base of the refactor of some old React Component Tests of mine.


## The problem

One year after adding at the ["Testing a Virtual List component with Cypress and Storybook"](../../sections/tools/cypress-and-storybook-exposing-component-from-story.md) chapter, and the ["Unit Testing React components with Cypress"](../../sections/tools/cypress-react-component-test.md) one, I felt terrible while realizing that my tests were almost unreadable. Many abstractions prevented me from carefully understanding what the test did in a while, resulting in a long time reading them. This is unacceptable friction.


## How to improve tests readability?

Kent Beck, the father of TDD, said

> Test code is NOT production code. It must be 1000x times simpler and smaller.

but how? What adds such friction to my tests? What worsens readability?

I will to analyze a bunch of my tests, reasoning about them, and propose a more straightforward approach. The component under test is a VirtualList mentioned in the previous articles.

## Test 1, low complexity

The next test is the simplest one: checking that the VirtualList renders nothing when nothing is passed. Following is the original test

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

Considering the simplicity of the test, is speaking about readability here bikeshedding? No, I've already some questions about it, such as:
- What does `getStoryItems` do?
- What does `createRenderItem` do?

I don't report `getStoryItems`'s code here, but it's just a function that generates an array containing X items in the form of `[{ id: 1, name: 'Item 1' }, { id: 2, name: 'Item 2' }, /* etc. */]`. Its purpose is to generate thousands of items easily, but I wrote it to ease writing Storybook's stories without tests in mind! Then, I reused it for the tests, but stories and tests have completely different needs and purposes!

At the same time, `createRenderItem` is a factory that generates the list items (some React components). Again, I made it such dynamic for Storybook, and tests weren't originally in my mind.

Both the functions are easy to read, but why am I forcing the reader to follow these abstractions? Are they needed? The answer is no.

Following is the simplified code of the test, following the differences.

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

In the simplified test, the most significant changes are:
1. I made `items` explicit instead of generating them through `getStoryItems`. The goal is immediately to clarify to the reader which are the items the VirtualList renders.
2. I removed the "unnecessary" constants. How tall are the items and the list don't make a difference if no items are rendered.
3. I removed the need for the `createRenderItem` factory. Generating components with height pre-set is useless here.

## Test 2, medium complexity

The next test is not complex. But the way I implemented it adds complexity when not needed. The goal is testing the base functionality of a VirtualList: rendering the visible items and not rendering the invisible ones.

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

Here things are getting weirder. Again, there are some simple abstractions, but the reader must:
- Comprehend the value of `visibleItemsAmount`.
- Comprehend what `visibleItems` contains, by reading `items.slice`. Again, one more complexity layer.
- Guess about what `itemsShouldBeVisible` does, or reading its code.
- Guess what `items[visibleItemsAmount]` contains.

Yeah, there are some comments. Yeah, I tried to create meaningful names. But I could simplify it a lot.

One more thing. Imagine this situation: the test is failing. It doesn't matter what you have done. This test is failing. You start debugging it, but you know that debugging a highly dynamic test (you don't know what `items`, `visibleItemsAmount`, `visibleItems`, `items[visibleItemsAmount]` contains upfront) is hard. You need to `console.log`/`cy.log`/`debugger`/breakpoint the test code to have an overall idea of the contents managed by the test, and then you can start debugging the VirtualList. Could I avoid future debugger these problems? Do I need the test to render 10.000 items?

Following is how I improved the test.

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

Also, here, the sole and relevant change is easing the future reader/debugger. You don't have to guess/calculate/log the value of the constants, nor what the abstractions do. The code is under your eyes, following the KISS principle (Keep It Simple, Stupid).

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

## Test 3, medium complexity

In the next test, the purpose is to test the VirtualList's `buffer` property that allows rendering some items even if they aren't visible yet.

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

The complexity comes from
1. Scrolling the list to test that the buffer acts on both sides.
2. When scrolling stops, I don't know in advance which items are visible and which not, that's why the content of `fastScrollVirtualList().then(() => { /* ... */ })` is dynamic.

I return on #2 later, but for this test, I cut the snake's head by removing #1: why do I need to test the entire `buffer` behavior here through a Cypress Component Test? I have a lot of unit tests that check that the internal VirtualList functions do their job. I don't need to re-test the same behavior again. Once `buffer` works, it works on both sides. The test benefit from this choice; now it's way simpler, containing a lot of comments that help the reader!

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

## Test 4, high complexity

Testing the selection is the hardest part of the VirtualList, because:
1. The VirtualList manages keyboard modifiers, allowing simple, additive, subtractive, and range selection.
2. The VirtualList is stateless: we need to wrap it with a stateful wrapper that stores the previous selection to test the additive, subtractive, and range selection.

My old test is very hard or read, because:
1. The stateful wrapper is declared in the body of the test itself, using the test's scope.
2. All the selections are checked out in a single long flow.

Step by step, let's simplify it.

### Moving the abstraction away

The first part of the old test is the following

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

It's pretty hard reading a test that starts with such a boilerplate. You get lost in a while, losing the critical parts of the test itself. Let's move it away from the test.

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

The background noise is still high, but you don't encounter it when you read the test. Then, do you notice that consuming the wrapper moved from
```tsx
mount(<WithSelectionManagement testHandleSelect={cy.stub().as('handleSelect')} />)
```
to
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
? The purpose is getting the code similar as much as possible to the previous, simpler tests. `SelectableList` is now more transparent, and its code is simpler because it does just one thing: storing the previous selection.

### Splitting the long test

Fasten the seatbelt: apart from the wrapper, the old test is the following

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

Looking at it now, the weird things are:
- `cy.then(() => expect(testItems).to.have.length.greaterThan(0))` since the items were generated dynamically, I bailed out in case of problems with the  external function.
- `cy.findByText(getItemText(testItems[0])).click()` is too much dynamic, what `testItems[0]` contains?
- The `fastScrollVirtualList().then(() => { /* ... */ }` content 😩
- The length of the test itself.
- It's not clear where testing a type of selection ends and where testing the next one begins.

Let's start by removing the need for such an extended test splitting a four-selection test into four ones.

#### Single selection

Guess what? Testing the simple selection doesn't need the wrapper at all 😊

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

The most relevant changes, compared to the first part of the old test:
- No need for the wrapper.
- No more dynamic values.
- More expressive assertions.
- It's a single-purpose test.

#### Additive and subtractive selection

Here we need to leverage the wrapper.

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

The huge difference here is avoiding scrolling because it doesn't affect the additive selection. Again explicit values, clarity, and brevity allow having a more straightforward and more expressive test.

Testing the subtractive selection follows the same pattern. I don't report it here.

#### Range selection

Compared to the previous selections, I think that the range selection **could be** affected by scrolling the list because some items that will be selected aren't rendered anymore. How could I make something that's dynamic by definition—scrolling the list—more static? Through manual tests and some comments.

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

The worst thing that could happen here is that the scrolling logics of the VirtualList scroll less or more when triggering the `wheel` event
```tsx
cy.findAllByTestId('VirtualList').first().trigger('wheel', {
  deltaX: 0,
  deltaY: 200,
});
```

but if it happens, the reader has a clear idea of what went wrong because of the comments 😊


## Related chapters

- 🔗 [Keep abstraction low to ease debugging the tests](/sections/generic-best-practices/test-code-with-debugging-in-mind.md)

<br/><br/>

*Crossposted by [NoriSte](https://github.com/NoriSte) on [dev.to](https://dev.to/noriste/from-unreadable-react-component-tests-to-simple-stupid-ones-3ge6).*
