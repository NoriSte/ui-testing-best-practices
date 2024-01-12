# UI 测试调试最佳实践

在转向 Cypress 之前，我通常使用 Puppeteer 编写 UI 测试。理解浏览器中发生的事情、了解正在运行的测试以及调试测试都不是简单的任务，因此我开始采取一系列解决方案来帮助我应对整个流程。

诸如 [Cypress](https://www.cypress.io/) 和 [TestCafé](https://devexpress.github.io/testcafe/) 的工具几乎使下面列出的最佳实践变得无关紧要，但除非你之前使用过 [Selenium](https://www.selenium.dev/) 或 [Puppeteer](https://pptr.dev/) 等工具进行测试，否则你不会意识到专为测试而设计的工具对简化生活有多么重要。

第零步是以非无头模式启动浏览器，然后...

## 在 console.log 中记录/显示测试的描述

由于在浏览器内部无法获得有关正在运行的测试的视觉反馈，请务必在浏览器控制台中记录测试的名称。在测试速度很快的情况下（少于 1 秒），这可能没有太多用处，但在测试时间较长或在使用 test.skip 和 test.only 进行测试时，这是有帮助的，可以对正在运行的测试进行双重检查。

在 Puppeteer 中，可以通过以下方式实现：

```js
test('Test description', async () => {
  await page.evaluate(() => console.log('Test description'));

  // ... the test code...
})
```

如果您需要更为显眼的反馈，甚至可以考虑在页面的左上角添加一个固定的 div，每个测试都会用自己的描述填充...

## Forward browser console.log to Node.js

With Puppeteer, a simple

```js
page.on('console', msg => console.log('BROWSER LOG:', msg.text()));
```

allows you to read both the test logs and the browser one in the same terminal window. Simple and effective.

### Launch the browser with the devtools already opened

Like in classic front-end development, opening the devtools after the page loading has already started could make you miss important stuff, especially in the network tab. While debugging your tests, launching the browser with the devtools already opened could save you precious time and info.

```js
const browser = await puppeteer.launch({
  headless: false,
  devtools: true
});
```

## Slow down the simulated user actions

A browser automation tool is fast as hell, this allows us to run a lot of tests in a bunch of seconds. While debugging, this could be a disadvantage, because you need to follow with your eyes what’s happening on the page. Slowing down **every action** could be counterproductive—because the whole test becomes slow— but usually, it is the easiest way to perform some rapid checks. In Puppeteer, there is a global setting to do that.

```js
const browser = await puppeteer.launch({
  headless: false,
  slowMo: 250, // slow down every action by 250ms
});
```

Some actions, like typing, allows you to add a more specific delay (that occurs on top of the global slowMo setting)

```js
await page.type('.username', 'admin', {delay: 10});
```

## Pause the test with a debugger statement

Again, like in standard web development, you could add a debugger statement to the code running on your page to “pause” the JavaScript execution. Please note: the statement is effective only if the devtools are opened in the controlled browser.

```js
await page.evaluate(() => {debugger;});
```

By clicking “Resume scripts execution” or bypressing F8 (debugger is a “flying” breakpoint) will resume the test execution.

## Increase the test timeout

Test runners like Jest, Jasmine, etc. have test timeouts. Timeouts are useful to terminate a test if something goes wrong and the tests never end. In UI testing, this behavior is cumbersome because you open the browser when the test starts and you close it when the test ends. In the normal test's lifecycle, a high timeout does not work because it results in a huge loss of time in case of failure, while a low one could “truncate” the test before it is finished.

Instead, you need long timeouts because you do not want the end of the test make close the browser while you are inspecting it. That’s why a 10-minutes timeout could be helpful while debugging the controlled browser.

Otherwise, you could...

## Avoid closing the browser when the test ends

When the tests start, you open the browser, then you close it when the tests end. Avoid closing it gives you the freedom of inspecting the front-end app without the fear of the test timeout. This is valid only when running the tests locally and the auto-close must be restored before running the tests on CI pipelines to avoid memory shortage due to the left open browser instances.

## Take screenshots

This is particularly helpful while running the tests in headless mode, in the phase when the tests are expected to be stable and to fail only in case of regressions. If a test fails, a lot of times a screenshot makes you to understand how the feature you are working on affected a previously working one. The most effective solution is to take a screenshot when the tests fail, otherwise, you could identify some checkpoints in your UI test and taking a screenshot at these steps.

## Assert frequently

A rule of thumb: if a test fails, it must drive you directly to understand what went wrong without re-launching the test and debugging it manually. Try to manually introduce bugs in your codebase (changing requests payload, removing elements, etc.) and check what the test reports. Is the error correlated to the bug you introduced? Who reads the failure could understand what needs to be fixed?

You need to add a lot of assertions to your test and it is totally fine! Unit tests take generally one single step and one or two assertions but UI tests are different, they have a lot of steps and hence you need a lot of assertions. Think of them as a series of uni tests where the previous test is necessary to create the ground for the second test, and so on.

## Use test.skip and test.only

It is one of the basics of every test runner but you never know: if you are not accustomed to using skip and only, start now! Otherwise, you are going to waste a lot of time, even if your test files are made by just two or three tests. Always run the least number of tests you are working on or you need to debug!

## Run the tests serially

If you are using Puppeteer in combination with Jest remember that Jest has a dedicated runInBand option that prevents it from spreading the execution of the tests over your CPU cores. Parallelizing the tests is perfect to fasten the execution but it is annoying when you need to follow the test actions with your eyes. runInBand runs the tests serially. Using it in combination with test.skip, test.only, and [jest-watch-typeahead](https://github.com/jest-community/jest-watch-typeahead) saves you a lot of debugging headaches.

## Keep your test code simple

Prefer duplication over abstraction. Strive to keep your tests code simple and easy to read. The more you debug UI tests, the more you realize how hard it could be. Your super-abstracted, completely DRYed, testing code becomes a pain as soon as you need to understand what is happening under the hood and which step is not working as expected.

More in general, tests are little scripts that must be two levels of magnitude simpler than the code they test, get them an ally, and not some more complex programs.

## Choose a made-on-purpose tool

All the above mentioned are valid solutions, but they have one thing in common: **they are workarounds**. Workarounds to the fact that the tool I used for the example, Puppeteer, has not been created for UI Testing, but for generic browser automation, then, with the help of some external utilities and consumed inside a test, Puppeteer could be used for UI Testing. Testing a web-app has different needs and requires different tools compared to just automating actions.

You should consider moving to Cypress or TestCafé if you need to write UI tests because they have been designed to simplify your testing life. How? With a large set of utilities and default behaviors and with a list of first-class solutions that allows you to understand and debug what is happening in the browser. Consider that all the Puppeteer **best practices mentioned** in this chapter... **Are completely useless with Cypress or TestCafé** 😉

The [Some UI testing problems and the Cypress way](../tools/ui-testing-problems-cypress.md) and [Front-end productivity boost: Cypress as your main development browser](./use-your-testing-tool-as-your-primary-development-tool.md) chapters include an overview of Cypress first-class utilities.

<br /><br />

_Crossposted by [NoriSte](https://github.com/NoriSte) on [dev.to](https://dev.to/noriste/ui-tests-debugging-best-practices-1eg3) and [Medium](https://medium.com/@NoriSte/ui-tests-debugging-best-practices-789c4ed4daf6?sk=c6056f124f40b15e09669e5839e9f814)._
