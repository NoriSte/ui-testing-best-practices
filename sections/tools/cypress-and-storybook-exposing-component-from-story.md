# Cypress + Storybook. Keeping test scenario, data and component rendering in one place.

_Russian version: [Cypress + Storybook. Хранение тестового сценария, данных и рендеринг компонента в одном месте](https://habr.com/ru/post/497544/)._

### One Paragraph Explainer

Many of us have chosen Cypress as a tool to test components hosted via Storybook/Styleguidist/Docz. [@NoriSte's example](./cypress-and-storybook.md) suggests creating some Storybook Stories, put components there and expose important data to the global variable in order to have access to the test. The nice approach actually, but the test becomes broken into the pieces between Storybook and Cypress.

Here I'd like to show how to go a little bit further and get the most out of executing JavaScript in Cypress. To see it in action, you may [download the source code](https://github.com/daedalius/article-exposing-component-from-storybook-to-handle-them-in-cypress) and execute then `npm i` and `npm test` in the console.

- You may expose the component reference from Storybook Story to test it however you wish in Cypress (without breaking testing logic into pieces).
- Cypress turned up so powerful for our team, so we do not have another utility that uses js-dom under the hood to test UI components.

## The task

Imagine that we are writing an adaptor for existing Datepicker component to use it across all company websites. We don't want to accidentally break anything, so we have to cover it by tests.

## Storybook

All we need from Storybook - an empty Story that saves a reference to the testing component in the global variable. In order not to be so useless this Story renders the single DOM node. This node will be our war zone inside the test.

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

Okay, we've finished with Storybook. Let's take a look at Cypress.

## Cypress

Personally, I like to get started with test cases enumeration. Seems we have next test structure:

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

Fine. We have to run this test in any environment. Open the Storybook, go directly to the empty Story by clicking at "Open canvas in new tab" button in the sidebar. Copy that URL and make Cypress visit it:

```jsx
const rootToMountSelector = '#component-test-mount-point'

before(() => {
  cy.visit('http://localhost:12345/iframe.html?id=datepicker--empty-story')
  cy.get(rootToMountSelector)
})
```

As you may guess, in order to test we are going to render all components states in the same `<div>` with `id=component-test-mount-point`. So that the tests do not affect each other, we must unmount any component here before the next test execution. Let's add some cleanup code:

```jsx
afterEach(() => {
  cy.document().then((doc) => {
    ReactDOM.unmountComponentAtNode(doc.querySelector(rootToMountSelector))
  })
})
```

Now we are ready to complete the test. Retrieve the component reference, render the component and make some assertions:

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

Do you see that? Nothing stops us from passing any props or data to the component directly! It's all in one place - in Cypress!

## Testing in a few steps with wrapper

Sometimes we'd like to test that component behaves predictably according to changing props.
Examine `<Popup />` component with `showed` props. When `showed` is `true`, `<Popup />` is visible. After that, changing `showed` to `false`, `<Popup />` should become hidden. How to test that transition?

Those problems are easy to handle in an imperative way, but in case of declarative React we need to come up with something.
In our team, we use an additional wrapper component with state to handle it. The state here is boolean, it responses to "showed" props.

```jsx
let setPopupTestWrapperState = null
const PopupTestWrapper = ({ showed, win }) => {
  const [isShown, setState] = React.useState(showed)
  setPopupTestWrapperState = setState
  return <win.Popup showed={isShown} />
}
```

Now we are about to finish the test:

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

> Tip: If such hook hasn't worked or you dislike calling the hook outside the component - rewrite the wrapper via simple class.

## Testing component methods

Actually, I've never written such a test. The idea has come up while writing this article. Probably it may be useful to test a component in a unit test style.

However, you may easily do it in Cypress. Just create a ref to the component before rendering. It is worth mentioning that the ref gives access to state and other elements of the component.

I've added the `hide` method to `<Popup \>` which makes it hidden forcibly (example for the sake of example). The following test looks like this:

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

## To sum it up: the roles of each participant

Storybook:

- Hosts Storybook Stories that contain bundled react components for test purposes.
- Provides a real non-synthetic environment to run tests.
- Each Story exposes one component in the global variable (to retrieve it in Cypress later).
- Each Story exposes a component mount point (to mount a component in test).
- Able to open each component in isolation in new tab.

> Pro-Tip: Please, run another instance of Storybook for your component library or pages.

Cypress:

- Contains and runs tests and Javascript.
- Visits isolated component Stories, retrieves component references from the global variable.
- Renders component according to testing needs (with any data or test conditions such as mobile resolution).
- Gives you super handy UI so you can see how your tests are going.

## Conclusion

Here I'd like to express my personal opinion and my colleagues' position about possible questions that may appear during the reading. Written below doesn't pretend to be true, may differ from reality and contain nuts.

### My test utils use js-dom under the hood. Do I limit myself?

- Js-dom is a synthetic environment. The separated DOM is not a real browser.
- It doesn't really work out to act with js-dom as it user does. Especially when it comes to simulating input events.
- How much confidence can you get from a written unit test if a component can be broken in CSS due to one incorrect z-index? If the component is tested by Cypress, you will see an error.
- You write unit tests blindly. But why?

### Should I choose the approach suggested?

- If you use tests as a development environment - definitely, Yes!
- If you look at tests as at **live** documentation - Yes.
- If you really write unit-tests to cover things that too close to implementation and React-lifecycle - ... I don't know. I haven't been writing such a test for a long time. Are you sure that the covered logic is component responsibility? Maybe that logic should be extracted and tested accordingly?

### Why not use cypress-react-unit-test then? Why do we need Storybook?

Probably, it is our future to test components. There will be no need to maintain a separate instance of the Storybook, all tests will be entirely under the responsibility of Cypress, the configuration will be simplified, etc.
But now the tool has [some](https://github.com/bahmutov/cypress-react-unit-test/issues/34) [problems](https://github.com/bahmutov/cypress-react-unit-test/issues/52) that make the environment provided incomplete for running tests. Hope that Gleb Bahmutov and the Cypress team will make it worked 🤞

_Crossposted by [daedalius](https://github.com/daedalius) on [Medium](https://itnext.io/cypress-storybook-keeping-test-scenario-data-and-component-rendering-in-one-place-c57b23cc1640) and on [habr.com (in Russian)](https://habr.com/ru/post/497544/)._
