# Choose a reference browser

<br/>

### One Paragraph Explainer

Everyone cares about cross-browser testing. We are used to manually testing everything on every browser because, we know, there are a lot of browser differences. Obviously, we'd like to have the same cross-browser support from the tool we use to test our front-end application... But the reality is that cross-browser support is very limited.

Don't take for granted that testing everything on every browser is always the best solution. Try to think about what you need to test before choosing a testing tool. Think that:

- **Selenium, and Puppeteer are generic automation tools**. They can be used as testing tools (there are a lot of plugins and modules that help you do that) but they are not created with testing in mind so they miss some integrated utilities that could simplify your life as a test writer.

- consider only **Cypress, Playwright, and TestCafé** because they are both tools created to **simplify the UI testing process**. Half of the best practices you should care in your tests are automatically managed by Cypress/Playwright/TestCafé out of the box. Choosing the right testing tool is all about finding the right trade-off. UI testing is hard by definition so, spend some time experimenting with the mentioned tools.

- think about what you need to test. Choose [TestCafé](https://testcafe.devexpress.com) if you need to test a particular mobile capability, but if you need to test that the forms and the buttons work you are less limited with your choice.

- take a look at the [Cypress Test Runner](https://docs.cypress.io/guides/core-concepts/test-runner.html#Command-Log), it's what makes Cypress outstanding and it's a precious ally during test development.

- take a look at what [Playwright offers in terms of debugging](https://playwright.dev/docs/trace-viewer-intro). Playwright is very fast and stable, and its DX improved a lot recently.

- cross-browser testing is usually related to visual testing (CSS browser differences) but this is not related to functional testing. Visual testing is made easy by a lot of dedicated plugins and tools. Take a look at [the dedicated chapter](../tools/visual-regression-testing.md) [Applitools](https://applitools.com) where we talk about some dedicated products that can be integrated with almost all the testing tools and work by uploading a snapshot of the page under test into their servers and render them.

You can also take a sneak peek at some differences between the various testing tools into the [Await, don't sleep](/sections/generic-best-practices/await-dont-sleep.md) chapter.

## Related chapters

- 🔗 [Combinatorial Testing](/sections/advanced/combinatorial-testing.md)
