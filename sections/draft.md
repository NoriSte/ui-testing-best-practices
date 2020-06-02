The _Topics list_ could be summarized in the next chapters

- typescrtipt in tests: pros and cons
- importance of assertions speakingness and having a lot of assertions in UI tests
- exporting constants from the code and import in the tests
- base tests on contents
- generic testing best practices
- visual regression test
- add more component testing articles (@makbeth has TestCafé+Storybook tests, re-try cypress-react-unit-test running `npm run component` on [this repo](https://github.com/cypress-io/cypress-example-todomvc-redux)) and improve the whole section. Update "Component tests vs (UI) Integration tests vs E2E tests" speaking about E2E tools + Storybook

Below you can find the original, detailed, _Topics list_.

---

### Topics list

- beginners? go to the dedicated section

- [ ] back-end related tests
  - [x] test request and response payloads
  - [x] test the server schema (or everything that can impact the front-end app like PostMan exports, Elastic Search mappings etc.). That should not be part of the front-end tests but consider that the app will be often broken by a server data change
  - [x] monitoring tests?
  - [ ] PACT
- [ ] testing strategy
  - [x] component vs integration vs e2e testing
  - [x] when you find a bug, write the test (that fails, it's important) and then fix the bug
  - [x] choose a reference browser (and if you need mobile browsers use TestCafè)
  - [x] avoid perfectionism, lot of UI interaction details are useful but they don't need to be tested (unless you have been proven that they bring value to your company)
  - [ ] if you use typescript in your app, use it in your tests too, the initial temptation of avoiding it will not last long
- [ ] ui testing best practices
  - [x] await, don't sleep
    - [x] wait for contents
      - [x] standard
      - [x] with custom selectors
    - [x] wait for network requests
    - [x] wait for front-end specific state
  - [ ] managing/stubbing websocket
  - [ ] assert frequently. An assertion is auto-explicative, an error isn't. We aren't unit-testing, every test has a flow and a lot of assertions
  - [ ] choose a made on purpose library
  - [ ] use the same front-end constants/functions (with an example with ReactIntl)
  - [x] UI testing framework as a development tool. Write tests step by step to avoid manual testing even during the development phase and to have them already written when you finished (easier with Cypress).
  - [ ] don't use the UI to reach the desired UI state. But remember to avoid writing one more app into your testing framework to consume the same resources
  - [ ] deterministic tests as wrote here https://docs.cypress.io/guides/core-concepts/conditional-testing.html#Error-Recovery
- [ ] in your app / practical advices
  - [ ] expose constants/functions. It makes tests more portable too. don't delay this process, by definition UI tests are slow, avoid them failing for a silly string
  - [ ] expose some shortcuts. Everything useful for letting the UI run faster (without compromising reliability) is welcome. let the front-end work for you
- [ ] base your tests on contents
  - [ ] ui testing is framework agnostic, base it on contents (the same consumed by the user)
  - [ ] pay attention on the pages that update frequently
  - [ ] a content-based failing test needs just a screenshot to be debugged, an attribute-based one needs the page to be inspected
  - [ ] if you can't rely on contents, use test-ids, never user classes of ids
  - [ ] avoid testing implementation details
- [ ] generic testing best practices
  - [ ] use the right assertion
  - [ ] use "only" and "skip"
  - [x] name the test file wisely
  - [ ] check the Yoni's post https://medium.com/@me_37286/yoni-goldberg-javascript-nodejs-testing-best-practices-2b98924c9347
  - [ ] don't share the state between tests
  - [ ] keep your tests simple with a low abstraction level
- [ ] component testing
  - [ ] use storybook (or a styleguide framework of choice) for components reporting all cases. In fact, they are tests too
  - [ ] add regression testing to storybook (or a styleguide framework of choice)
- [ ] performance
  - [ ] always choose the "simplest"/fastest test. dom-ui-testing is faster then Cypress etc.
- [ ] UI test debugging (mark every single tip necessary/unnecessary with existing frameworks) (the problems are partially explained in the Cypress chapter, list these problems too. It could become a "how to debug a test without Cypress")
  - [ ] make screenshots
  - [ ] launch the browser in non-headless mode
  - [ ] launch the browser with the devtools already opened
  - [ ] slow down the browser actions
  - [ ] increase the test timeout
  - [ ] avoid closing the browser when the test ends
  - [ ] console.log the name of the test
- [ ] frameworks/tools
  - [x] Cypress
  - [ ] TestCafé
- [ ] beginners
  - [ ] what is UI testing
    - [ ] why is it so important
    - [ ] headless browser
    - [ ] use it while studying
    - [ ] automate repetitive and annoying
    - [x] use testing frameworks as development tools
    - [ ] data scraping (take care that some sites will block you because of the high number of login requests etc.)
    - [ ] avoid regression
    - [ ] avoid manually testing
  - [ ] the other kind of tests
    - [ ] unit
    - [ ] integration
    - [ ] regression
    - [ ] snap shot testing
    - [ ] the testing pyramid (standard and the kent's one)
    - [ ] Always use the simplest kind of test
  - [x] Why it's hard to do UI testing (actually mixed with the Cypress section)
  - [x] tests as a documentation tools
- [ ] advanced
  - [ ] cypress code coverage
  - [ ] cucumber and gherkins deserve some words
- [ ] a11y?
- [ ] take a look at every TODO in the various contents
- [ ] link every section to each other
- [ ] consider adding some links to a-z testing
- [ ] take a look at the ReactJSDay course book and add new contents
- [ ] explain why, in Cypress, custom commands can't have all the assertions we'd like (no retrieval from failing error)

- take a look at https://slides.com/noriste/milano-frontend-ui-testing-best-practices (and the React Testing course too) to check if
  some contents are missing

For contributors, write:

- that a lot of missing content could be applied and you ask for contributors
- that your English could need a check
- that you'd like to hear from other experts, you have not years of experience with every tool
- tool creators could help a lot, too
