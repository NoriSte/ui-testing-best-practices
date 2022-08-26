# Unit Testing React components with Cypress

_**UPDATE**: [Cypress 10 is out with Component Testing integrated with E2E testing](https://www.cypress.io/blog/2022/06/01/cypress-10-release/), please check it out and ignore all the configuration steps reported below since they are outdated!_

---

_**UPDATE**: [Cypress 7 is out with a brand-new Component Test](https://docs.cypress.io/guides/component-testing/introduction#What-is-Component-Testing) support, check it out! And other exciting news is on the way thanks to [Storybook 6.2 release](https://twitter.com/NoriSte/status/1378204109841571840)!_

---

Now that unit testing React component is possible with Cypress, this is an extending chapter of the [Testing a component with Cypress and Storybook](./cypress-and-storybook.md) one.

The goal of the previous chapter was to run some experiments in the **React Component Testing world**, a really important topic nowadays.

The motivations were pretty simple:

- you probably already have [Storybook](https://storybook.js.org/) in action in your team (if not, consider adding it!)

- you could be not familiar with testing components with [Testing Library](https://testing-library.com/) or you could be biased about JSDom or you could want to test your UI components in a real browser, not in a simulated DOM environment

- you could be familiar with [Cypress](https://www.cypress.io/) or [TestCafé](https://devexpress.github.io/testcafe/) (if not, consider them for your UI tests) and you could want to use just a single tool for your tests

And the approach was simple too:

- exposing the story’ props to the testing tool, used to control the rendered component

- pick up them from Cypress/TestCafé, automating user actions and asserting about the contents of the props

But there were **some caveats**…

- performance: in the [chapter](https://github.com/NoriSte/all-my-contributions), I put some extra-efforts to minimize the impact of story switching slowness

- **testing and stories coupling**: since Storybook is consumed even by Cypress, stories are going to be accountable not only for sharing the design system across the team but for the component tests too

- **callback testing got tough**: checking the params and the calls of the callback props is difficult

Some of the problems of my experiment could be mitigated by [daedalius approach](./cypress-and-storybook-exposing-component-from-story.md) but the solution is not optimal yet, but then…

## Cypress 4.5.0 has been released

On April, 28th, Cypress 4.5.0 has been released, the only released feature is the following

> Cypress now supports the execution of component tests using framework-specific adaptors when setting the experimentalComponentTesting configuration option to true. For more details see the [cypress-react-unit-test](https://github.com/bahmutov/cypress-react-unit-test) and [cypress-vue-unit-test](https://github.com/bahmutov/cypress-vue-unit-test) repos.

What does it mean? That Cypress can now directly mount a React component giving the **[cypress-react-unit-test](https://github.com/bahmutov/cypress-react-unit-test)** a new birth! Before Cypress 4.5.0 release, the plugin was pretty limited but now it has first-class support! In fact, the cypress-react-unit-test is now rock-solid and a meaningful plugin.

## Testing the VirtualList component: second episode

The component is always the same, the VirtualList, read more about it in [the previous chapter](./cypress-and-storybook.md). We need to set up both the [cypress-react-unit-test](https://github.com/bahmutov/cypress-react-unit-test) and the TypeScript conversion (the component is written in TypeScript, it is part of a [Lerna](https://github.com/lerna/lerna) monorepo, and it is compiled with Webpack). Both the steps are straightforward but if the plugin has an [installation-dedicated section in its documentation](https://github.com/bahmutov/cypress-react-unit-test#install), the TypeScript compilation could not be obvious because there are, outdated or partial, a lot of different approaches and resources.
The most concise yet effective solution is [André Pena’s one](https://stackoverflow.com/a/60017105/700707), so all I had to do is:

- adding a _cypress/webpack.config.js_ file

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

- adding a _cypress/tsconfig.json_ file

```json
{
  "extends": "../tsconfig.json",
  "compilerOptions": {
    "types": ["cypress", "cypress-wait-until"]
  }
}
```

please note that:

- the ../tsconfig.json file is the same used by the React app

- [cypress-wait-until](https://github.com/NoriSte/cypress-wait-until) is not mandatory but I use it a lot and it is one of the most installed plugins for Cypress

The above transpiling-related files, along with the following cypress.json file

```json
{
  "experimentalComponentTesting": true,
  "componentFolder": "cypress/component"
}
```

are enough to start playing with a _cypress/component/VirtualList.spec.tsx_ test! From the previous chapter, the first test was the standard rendering, the _“When the component receives 10000 items, then only the minimum number of items are rendered”_ test, et voilà:

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

Compared to the Storybook-related chapter:

- the

```js
/// <reference types="Cypress" />
/// <reference types="cypress-wait-until" />
```

at the beginning are needed to let VSCode correctly leverage TypeScript suggestions and error reporting

- we use cypress-react-unit-test’ mount API to mount the component, nothing especially new if you are used to the [Testing Library APIs](https://testing-library.com/docs/react-testing-library/api)

Nothing more, the Cypress test continues the same as the Storybook-related one 😊

### Callback Testing

Porting all the tests from the [previous chapter](./cypress-and-storybook.md) is quite easy, what was missing is the callback testing part of the “selection test”.

Creating a _WithSelectionManagement_ wrapper component that renders the _VirtualList_ one and manages items selection is quite easy and we can pass it our stub and assert about it

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

Please refer to the full SinonJS (wrapped and used by Cypress) [Stub](https://sinonjs.org/releases/v9.0.2/stubs/)/[Spy](https://sinonjs.org/releases/v9.0.2/spies/) documentation for the full APIs.

## Conclusions

Take a look at the result in [this video](https://youtu.be/yVL-yRT5hFQ). The test lasts now less than seven seconds, without depending nor loading Storybook, leveraging first-class Cypress support.

What’s next? The [cypress-react-unit-test](https://github.com/bahmutov/cypress-react-unit-test) plugin is quite stable and useful now, a whole new world of experiments is open and a lot of small-to-medium projects could choose to leverage Cypress as a single testing tool.

<!-- TODO: in the end, decide if you want to move all the resources to a common chapter too -->

<br /><br />

_Crossposted by [NoriSte](https://github.com/NoriSte) on [dev.to](https://dev.to/noriste/unit-testing-react-components-with-cypress-5fk2) and [Medium](https://medium.com/@NoriSte/unit-testing-react-components-with-cypress-4d4cf8cd59a0)._
