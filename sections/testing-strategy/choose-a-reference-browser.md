# Choose a reference browser

<br/>

### One Paragraph Explainer

Everyone cares about cross-browser testing. We are used to manually testing everything on every browser because, we know, there are a lot of browser differences. Obviously, we'd like to have the same cross-browser support from the tool we use to test our front-end application... But the reality is that cross-browser support is very limited.

The next table reports the cross-browser support for the major testing/automation tools.

| Tool       | Chrome | Firefox | IE11 | Edge | Safari | iOS Safari | Android Chrome |
|------------|--------|---------|------|------|--------|------------|----------------|
| [Cypress](https://www.cypress.io)    | ✅     | ⌛<sup>[1](#footnote1)</sup>     | ⌛️<sup>[1](#footnote1)</sup>   | ⌛️<sup>[1](#footnote1)</sup>   | ⌛     | ⌛️<sup>[1](#footnote1)</sup>        | ⌛️<sup>[1](#footnote1)</sup>             |
| [TestCafe](https://testcafe.devexpress.com)   | ✅     | ✅️     | ✅   | ✅   | ✅     | ✅️        | ✅️             |
| [Puppeteer](https://pptr.dev)  | ✅     | ⌛<sup>[2](#footnote2)</sup>     | ⬜️   | ⬜️   | ⬜     | ⬜️        | ⬜️             |
| [Selenium](https://www.seleniumhq.org)   | ✅     | ✅     | ✅   | ✅   | ✅     | ✅️<sup>[3](#footnote3)</sup>        | ✅<sup>[3](#footnote3)</sup>             |

<br />
Anyway: don't take for granted that testing everything on every browser is always the best solution. Try to think about what you need to test before choosing a testing tool. Think that:

- both **Selenium and Puppeteer are generic automation tools**. They can be used as testing tools (there are a lot of plugins and modules that help you do that) but they are not created with testing in mind so they miss some integrated utilities that could simplify your life as a test writer
- consider only **Cypress and TestCafe** because they are both tools created to **simplify the UI testing process**. Half of the best practices you should care in your tests are automatically managed by Cypress/TestCafe out of the box. Choosing the right testing tool is all about finding the right trade-off. UI testing is hard by definition so, spend some time experimenting with both Cypress and TestCafe
- think about what you need to test. Choose [TestCafe](https://testcafe.devexpress.com) if you need to test a particular mobile capability, but if you need to test that the forms and the buttons work you are less limited with your choice
- take a look at the [Cypress Test Runner](https://docs.cypress.io/guides/core-concepts/test-runner.html#Command-Log), it's what makes Cypress outstanding and it's a precious ally during test development
- cross-browser testing is usually related to visual testing (CSS browser differences) but this is not related to functional testing. Visual testing is made easy by a lot of dedicated plugins and tools. Take a look at [Applitools](https://applitools.com) and [Percy](https://percy.io), both can be integrated seamlessly in every testing tool and they work by uploading a snapshot of the page under test into their servers and render them

You can take a sneak peek at some differences between the various testing tools into the [Await, don't sleep](/sections/generic-best-practices/await-dont-sleep.md) chapter.

<br />

<a name="footnote1">1</a>: crossbrowser support [is on the Cypress roadmap](https://github.com/cypress-io/cypress/issues/310)
<br />
<a name="footnote2">2</a>: Firefox support for Puppeteer [is not complete yet](https://aslushnikov.github.io/ispuppeteerfirefoxready/)
<br />
<a name="footnote3">3</a>: Selenium supports mobile browser through [Appium](http://appium.io/docs/en/writing-running-appium/web/mobile-web/)
