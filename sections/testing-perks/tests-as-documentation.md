# Software tests as a documentation tool

<br/><br/>

### One Paragraph Explainer

Documenting is generally hard, it requires precise and meticulous work and it requires that all the members of the team understand and value writing good documentation. Documenting is selfless and it is **helpful for both other developers and the future you**.

Testing methodologies are a great way not only to be sure that we code what the project needs, not only to grant we do not introduce regressions but to document the code and the user flows too.

The perks that come from using the tests as a documentation tool are:

- **the documentation is coupled** with what the code has been created for: all the UI tests should be written from the user perspective and their descriptions either. Watching what the user is able to do with the project is a really effective way to know the project from a functional point of view.<br>Every codebase is composed of thousands of small pieces of code and sometimes **it could be hard to connect all the dots**. The tests could allow a general understanding of the project and even a lot of technical details.

  <!-- TODO: add a link to the blackbox testing chapter -->

- you do **not rely on the historical memory** of some employees: a lot of times you end up asking to some employees that know the project and remember some particular edge cases. A good test suite can reduce a lot the needed for this kind of knowledge and avoid every fresh developer to add regressions with a few lines of code.

- at the same time, the **handover and onboarding** phases become quite easy.

Bonus point: if you leverage the [Gherkin](https://cucumber.io/docs/gherkin/reference/) syntax, the documentation effectiveness is increased even for some less-technical people like a QA team.

Please, keep in mind that:

- test descriptions must be clear even for developers that do not know the project context the same way as you

- the re-used test functions, fixtures, etc. must have meaningful names. A `registration-success.json` fixture used for both the sign-up and the login tests could mislead the future reader and make historical knowledge needed. Remember that relying on historical knowledge is always negative for a codebase that must survive the developers' turnover.

- in general, UI Testing plays a fundamental role in a front-end application, they are the only ones that document the real goals the user is expected to be able to accomplish.

- the code of the tests must as simple as possible. Simple to be read, condition-free, with a low-abstraction level, with a good level of logging, etc. Always remember that **the tests must reduce the cognitive load of reading and understanding the code**, hence their complexity should be an order of magnitude lower compared to the code to be understood. This improves the deepening process that a developer must go through just after have watched the tests in the automated browser.

  <!-- TODO: add a link to the chapter that speaks about why the test code must be simple -->

- "connect" the code with the tests: if a user flow is quite long, it could be useful to share some "steps" (with some comments) between the source code and the code of the tests. Something like `/** #1 \*/`, `/** #2 \*/`, etc.

- UI tests are not the only ones: having more low-level tests for parts of the code that could be hard to be understood is a great way to describe the code expected behaviors.
