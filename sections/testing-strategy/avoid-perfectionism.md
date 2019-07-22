# In the beginning, avoid perfectionism

<br/>

### One Paragraph Explainer

Testing really changes the way you work but, like everything, it requires a bit of experience to get the best of it. In the beginning, avoid perfectionism at all. Why?

- tests are little programs after all. Perfectionism could drive you to write **extremely complex tests** before knowing how to manage the different testing contexts.

  Tests complexity is a big enemy because debugging a failing test is harder than debugging a failing application. And complex tests make you lose the advantage of testing practices themselves, make you lose a lot of time, and inevitably make you exclude them sooner or later. **If it happens to you don't give up**, it's the same for a lot of testing beginners (it was the same for me, that's why I started writing this repo 😊) and do not be afraid of asking your colleagues or publicly to other developers.

- false negatives: perfectionism-driven tests lead to a lot of false negatives. A false negative is a situation where your application works as expected but the test fails.

  **False negatives are really de-motivating** at the beginning because you started writing tests to have an ally in checking the application status... But you ended up with another application to maintain with no help from the tests. If you realize you are fighting with false negatives, stop yourself, restart studying and ask for help!

- tests utility: successful tests drive you directly to the problem when they fail. The right assertions and [deterministic events](/sections/generic-best-practices/await-dont-sleep.md) make your tests robust and, super important, useful if they fail. At the opposite, too much assertions/checks could make your tests brittle because of their uselessness

What do you mean for perfectionism? I mean checking every front-end detail. Your working experience allows you to write complex user interactions but, in the beginning, your limited testing experience do not allow you to test all of the interactions profitably. Start testing the easiest things like
- is my page loading correctly?
- do the menu buttons works?
- can the user fill the form and reach the thank you page?

and forget, in the beginning, to test things like
- conditional data loading
- complex form rules
- uncontrolled (third party) integrations
- element selectors

<br />

A beginning todo list to avoid falling into the perfectionism trap could be:

- choose the simplest thing to test (something that's useful for the users)
- think about it from the users perspective. Remember that the users care about contents and functionalities, not about selectors and internal application states
- write your test
- run it more than once to be sure that it's stable
- when it succeeds, insert a bug into the front-end app that breaks it and checks that the test fails. Then, remove your made-on-purpose bug
- run the test both in headless and non-headless mode
- think about, based on your experience (ask your colleagues too), what are the reasons that could break the front-end application from the perspective of what you're testing
- simulate the different front-end failures (kill the server, insert other bugs) and check if the test gives you enough feedback to understand what failed
- do that for just two or three kinds of failures, remember that your limited experience could drive you to test the wrong things
- then, move to another thing to test and repeat all the previous steps

Software testing is an amazing journey and the goal of this repo is helping you avoid the most common pitfalls.

The suggested flow is just one of the possible approaches. I know that everything is subjective and please, open a PR for every suggestion!
