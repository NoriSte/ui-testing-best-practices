
# One long E2E test or small, independent ones?

<br/><br/>

### One Paragraph Explainer


While speaking about testing a CRUD app, how should we organize the "create", "modify", and "delete" E2E tests?

The complete list of options are:

1. To have **three small E2E tests dependent on the execution order** (test B takes for granted that test A run) - The only bad solution, I'm going to explain why.
2. To have **three small E2E tests independent from the execution order** (test B works regardless from whether test A was launched or not) - Theoretically, the best solution. Still, it requires a lot of boilerplates also to be fast.
3. To have **one extended E2E test** that does everything - A good tradeoff for the case presented in this article.

It depends, and most of the problems I present are related to the implicit issues of the E2E tests, a strong signal that we should write only a few of them. As a Front-end Engineer, I strongly prefer to invest my time in server-free tests, not E2E ones. Go ahead, and you will understand why.

Please note: this results from working on Hasura's Console E2E tests.

<br/><br/>



## 1 - To have three small E2E tests dependent on the execution order (test B takes for granted that test A run)

The test flow would be something like this:

1. START (*the application state is empty*)
2. Test 1: create the entity
3. Test 2: modify the entity
4. Test 3: delete the entity
5. END (*the application state is empty*)

In this case, the tests are not independent but based on execution order. To test a CRUD flow, the three primary tests are "create an entity", "modify an entity", "delete the entity". The second test ("modify the entity") takes for granted that when it starts, the application state is okay because it runs after the "create the entity" one. "delete the entity" must run after the "modify the entity" too, etc.

Coupling multiple tests together is an anti-pattern because of:

- **False negatives**: The tests will fail in a row once one fails.
- **Hard to debug**: understanding the root of a failure is more complicated because of higher ambiguity. Did the test fail because of its code? Or because the state of the previous test changed? Then, you have to debug two tests when one fails.
- **Hard to debug** (again): the developers waste a lot of time because they can not run a single test nor use `skip` and `only` to launch a portion of them.
- **Hard to refactor**: The tests cannot be moved elsewhere. If the code of the tests becomes too long, too complex, etc., you cannot move it to a dedicated file/directory because it depends on the previous one.
- **Hard to read**: The readers cannot know what a test does because they must also know the previous tests. You have to read two tests instead of one, which is not good.

I do not recommend writing tests coupled this way, but I want to include them to be sure you realize why.

## 2 - To have three small E2E tests independent from the execution order

To get every test independent, every test should create the application state that it needs to run, then clear it after completion. The flow presented in the previous chapter should become something like (in *italic* the new steps compared to the original create->modify->delete three tests in a row)

1. START (*the application state is empty*)
2. Test 1: create the entity
    1. ***BEFORE****: Load the page (the application state is empty)*
    2. create the entity
    3. ***AFTER****: Delete the *entity* (the application state is empty)*
3. Test 2: modify the entity
    1. ***BEFORE****: create the *entity* (through APIs)*
    2. ***BEFORE****: Load the page (the application state is empty)*
    3. modify the entity
    4. ***AFTER****: Delete the *entity* (through APIs, the application state is empty)*
4. Test 3: delete the entity
    1. ***BEFORE****: create the *entity* (through APIs)*
    2. ***BEFORE****: Load the page (the application state is empty)*
    3. delete the entity
    4. ***AFTER****: Delete the action (the application state is empty)*
5. END (*the application state is empty*)

By doing so, every test is independent. Please note that the before and after actions are done directly by calling the server APIs. Doing them through the UI would be too slow.

Anyway, the presented approach's problem is that **tests become slower** because every test creates the entity, and every test visits the page. When the application takes 10 seconds to load (it was initially the case of Hasura's Console), reloading the app is a problem.

To get the best of both worlds (independent but fast tests), we should evolve the above flow to

- Exploit the previous test's application state.
- But also create the needed application state if no tests have run.

Something like (in *italic* the new steps compared to the flow presented in the previous chapter)

1. START (*the application state is empty*)
2. Test 1: create the entity
    1. ***BEFORE****: Does the *entity* exist?*
        1. *NO: it's ok!*
        2. *YES: delete the entity (through APIs)*

    2. **BEFORE**: Load the page (the application state is empty)
    3. create the entity
3. Test 2: modify the entity
    1. ***BEFORE****: Does the *entity* exist?*
        1. *YES: it's ok!*
        2. *NO: create the entity (through APIs)*
    2. ***BEFORE****: Does the *entity* already includes the change the test is going to make?*
        1. *YES: it's ok!*
        2. *NO: modify the entity (through APIs)*
    3. ***BEFORE****: Are we already on the correct page?*
        1. *YES: it's ok!*
        2. *NO: load the page*
    4. modify the entity
4. Test 3: delete the entity
    1. ***BEFORE****: Does the entity exist?*
        1. *YES: it's ok!*
        2. *NO: create the entity (through APIs)*
    2. ***BEFORE****: Are we already on the correct page?*
        1. *YES: it's ok!*
        2. *NO: load the page*

5. delete the entity
6. END

Now, if you run all the tests in a row, each of them leverages the existing application state. If you run just the "modify the entity" for instance, it creates whatever it needs, then runs the test itself.

Now we have both test independence and test performance! Cool!

Well... Did you notice the amount of code we need to write? The [cypress-data-session](https://github.com/bahmutov/cypress-data-session) plugin comes in handy but

1. You have a lot of cypress-data-session related boilerplate
2. You have to maintain, in the E2E tests, a lot of API calls that could go out of sync with the ones used in the main application in a while

Here is an example of the cypress-data-session related boilerplate (coming from the Hasura Console codebase)

```ts
import { readMetadata } from '../services/readMetadata';
import { deleteHakunaMatataPermission } from '../services/deleteHakunaMatataPermission';

/**
 * Ensure the Action does not have the Permission.
 *
 * ATTENTION: if you get the "setup function changed for session..." error, simply close the
 * Cypress-controlled browser and re-launch the test file.
 */
export function hakunaMatataPermissionMustNotExist(
  settingUpApplicationState = true
) {
  cy.dataSession({
    name: 'hakunaMatataPermissionMustNotExist',

    // Without it, cy.dataSession run the setup function also the very first time, trying to
    // delete a Permission that does not exist
    init: () => true,

    // Check if the Permission exists
    validate: () => {
      Cypress.log({ message: '**--- Action check: start**' });

      return readMetadata().then(response => {
        const loginAction = response.body.actions?.find(
          action => action.name === 'login'
        );

        if (!loginAction || !loginAction.permissions) return true;

        const permission = loginAction.permissions.find(
          permission => permission.role === 'hakuna_matata'
        );

        // Returns true if the permission does not exist
        return !permission;
      });
    },

    preSetup: () =>
      Cypress.log({ message: '**--- The permission must be deleted**' }),

    // Delete the Permission
    setup: () => {
      deleteHakunaMatataPermission();

      if (settingUpApplicationState) {
        // Ensure the UI read the latest data if it were previously loaded
        cy.reload();
      }
    },
  });
}

```

and here is an example of the API call to create the entity (coming from the Hasura Console codebase)

```ts
/**
 * Create the Action straight on the server.
 */
export function createLoginAction() {
  Cypress.log({ message: '**--- Action creation: start**' });

  cy.request('POST', 'http://localhost:8080/v1/metadata', {
    type: 'bulk',
    source: 'default',
    args: [
      {
        type: 'set_custom_types',
        args: {
          scalars: [],
          input_objects: [
            {
              name: 'SampleInput',
              fields: [
                { name: 'username', type: 'String!' },
                { name: 'password', type: 'String!' },
              ],
            },
          ],
          objects: [
            {
              name: 'SampleOutput',
              fields: [{ name: 'accessToken', type: 'String!' }],
            },
            {
              name: 'LoginResponse',
              description: null,
              fields: [
                {
                  name: 'accessToken',
                  type: 'String!',
                  description: null,
                },
              ],
            },
            {
              name: 'AddResult',
              fields: [{ name: 'sum', type: 'Int' }],
            },
          ],
          enums: [],
        },
      },
      {
        type: 'create_action',
        args: {
          name: 'login',
          definition: {
            arguments: [
              {
                name: 'username',
                type: 'String!',
                description: null,
              },
              {
                name: 'password',
                type: 'String!',
                description: null,
              },
            ],
            kind: 'synchronous',
            output_type: 'LoginResponse',
            handler: 'https://hasura-actions-demo.glitch.me/login',
            type: 'mutation',
            headers: [],
            timeout: 25,
            request_transform: null,
          },
          comment: null,
        },
      },
    ],
  }).then(() => Cypress.log({ message: '**--- Action creation: end**' }));
}
```

So, having independent tests is essential, but it comes with a cost.

That's why, for this specific problem, I then opted for the last option...

## 3 - To have one extended E2E test that does everything

Pros: a lot of boilerplate files can be removed.

Cons: Working with the tests becomes slower (you cannot launch only the third test anymore)

Compared to the boilerplate we need to write and the code we need to maintain, it is worth unifying them. After all, the specific CRUD flow I was working on took ~20 seconds.

1. START (*the application state is empty*)
2. Test: CRUD
  1. ***BEFORE****: Delete the entity if it exists (the application state is empty)*
  2. ***BEFORE****: Load the page*
  3. create the entity
  4. modify the entity
  5. delete the entity
  6. ***AFTER****: Delete the entity if it exists (the application state is empty)*
3. END (*the application state is empty*)

And at the same time, it makes cypress-data-session useless. Hence one less dependency to keep updated.

## Conclusions

Working with E2E tests is hard. Dealing with real data, real application state to clear, etc., has a cost. I know that E2E tests are the only ones that give complete confidence, but as a Front-end Engineer (remember, I'm not a QA Engineer), I strongly prefer to work with server-free tests.


## Related chapters

- 🔗 [Approach the testing pyramid from the top!](/sections/beginners/top-to-bottom-approach.md)
- 🔗 [Use your testing tool as your primary development tool](/sections/generic-best-practices/use-your-testing-tool-as-your-primary-development-tool.md)
<br/><br/>

*Crossposted by [NoriSte](https://github.com/NoriSte) on [dev.to](https://dev.to/noriste/decouple-the-back-end-and-front-end-test-through-contract-testing-112k).*
