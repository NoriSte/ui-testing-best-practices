# Found a bug? Write the test, then fix it

<br/>

### One Paragraph Explainer

So, you have found a bug in your front-end application and you have already debugged it. You can reproduce it systematically and you're ready to fix it. A testing-oriented mind must pass through these steps:
- identify the expected behavior
- write a test that aims to consume the front-end application the correct way
- the test must fail, obviously, because the bug does not allow the user to perform his task
- fix the bug
- check that the test now passes

What are the advantages of this approach? Why should you write a test? I know that fixing the bug without a test could seem faster but consider that:
- as usual, your testing tool is faster than you to reach the state of the application that shows the bug (see the [Use your testing tool as your primary development tool](/sections/generic-best-practices/use-your-testing-tool-as-your-primary-development-tool.md) chapter)
- sometimes you think that you're able to reproduce the bug systematically, but it's not always true. Writing a test that unearths the bug make you 100% sure that the bug is reproducible **excluding a lot of deviant variables** like existing sessions, caches, service workers, browser extensions, browser version, etc. that could influence your confidence. Sometimes you discover that you did not identify the bug correctly
- at the same time, when the test passes thanks to your fix, you are really sure that your solution works as expected. The same variables that could influence your bug identification process could mine the false sense of the job's effectiveness
- **with a test, the bug is fixed forever!** The test will be run thousands of times and let you sleep well about that forever
- a successful test could be used as a proven track of the work you've done

Last but not least: be sure that the test you write fails at the beginning! And that it fails because of the bug! The test is not useful only to reproduce the bug and check it visually, but it must put you in the position of having positive feedback after the bug fixing process. **A bug-related test that does not fail is really dangerous** because you can think that you've made a good job when the reality is that you have not reproduced the bug correctly since the beginning. As a general rule: a broken flow must have a broken test, a successful test must be related to a working app.
