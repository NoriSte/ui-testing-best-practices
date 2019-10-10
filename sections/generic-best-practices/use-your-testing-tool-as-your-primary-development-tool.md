# Use your testing tool as your primary development tool

<br/><br/>

### One Paragraph Explainer


An example speaks itself. Let's say you're developing an authentication form, probably, you:
- code the username input field
- **try it manually** in the browser
- code the password input field
- **try it manually** in the browser
- code the submit button
- code the XHR request management

then, you can't go ahead because, without changing the source code, you need that a stubbed/mocked server responds to the app XHR requests. **Then you start writing an integration test** that:
- fill the username input field
- fill the password input field
- click the submit button
- check the XHR request
- stub the XHR response
- check the feedbacks
- re-do it for every error flow
- code the error flow management
- re-check the tests

Take a look at the first test steps, they are **the same we made manually** while coding the authentication form. Then, we stub the
server responses and check the final behavior of the form (with success/failure responses).

This working flow could be easily improved if we write the test alongside the form itself (TDD
developers are already trained to do that):
- **code the username input field** *
- **write the test to fill** the username input field *
- code the password input field
- update the test to fill the password input field
- code the submit button
- update the test to click the submit button
- stub the XHR response into the test
- code the XHR request management
- check the feedbacks
- code the error flow management
- update the test to check the error flow
- re-do the above steps for every single error flow

\* please note that the first and the second step could be inverted if you want to apply a strict TDD approach

What are the most important advantages of doing that?
- you **avoid (almost) completely manually testing** your app
- you leverage the speed of your testing tool, it fills the form at a blazing speed and let you **save
  a lot of time**
- you don't have to write the test after you coded the form (again, TDD developers already avoid it)
  that, only at the first approaches, could seem an annoying task
- you completely **avoid putting some temporary states into your source code** (input field default
  values, fake XHR responses)
- you test your app directly with a real network response (remember that your app does not know that
  the network request is stubbed by the testing tool)
- the test is relaunched every time you save the test file
- you can leverage both the Chrome DevTools and the framework-specific devtools

How to leverage the **existing Development Tools**?<br>
Well, you can do that in
almost every testing tool but Cypress stands out for this kind of goal. Cypress has a dedicated
Chrome user that's persisted across all your tests and all your projects. Doing so, Cypress allows
you to have a real testing-dedicated browser with your favorite extensions, even if you use the
same Chrome version you use to browse.<br>
Mix it with a great UI and you are ready to start developing all your app directly with Cypress.

Below you can find some screenshot of the Cypress UI to show you how much easy is using it as a
primary development tool.

<br>

**Browser Selection**
![Cypress browser
selection](../../assets/images/use-your-testing-tool-as-your-primary-development-tool/browser-selection.png
"Cypress browser selection")

**The Cypress-controlled browser DevTools**
![Cypress browser
devtools](../../assets/images/use-your-testing-tool-as-your-primary-development-tool/devtools.jpg
"Cypress browser devtools")

**Cypress [Skip and Only UI plugin](https://github.com/bahmutov/cypress-skip-and-only-ui)** that allows you to add some `.only` or `.skip` to the tests directly from the Cypress UI.
![Cypress Skip and Only
UI](../../assets/images/use-your-testing-tool-as-your-primary-development-tool/skip-and-only.gif
"Cypress Skip and Only UI")

**Cypress [Watch and Reload plugin](https://github.com/bahmutov/cypress-watch-and-reload)** that allows you to re-run the cypress tests on every source code compilation.


If you want to see the React/Redux devtools in action with the Cypress controlled browser you can use the [cypress-react-devtools](https://github.com/NoriSte/cypress-react-devtools) repository.

<br /><br />

*Crossposted by [NoriSte](https://github.com/NoriSte) on [dev.to](https://dev.to/noriste/front-end-productivity-boost-cypress-as-your-main-development-browser-5cdk) and [Medium](https://medium.com/@NoriSte/front-end-productivity-boost-cypress-as-your-main-development-browser-f08721123498).*
