# Email Testing

<br/><br/>

### One Paragraph Explainer

Email testing is [critical for business success](https://www.industrialmarketer.com/why-email-testing-is-critical-for-email-marketing-success/) and [boosts email performance](https://litmus.com/blog/3-reasons-why-email-testing-boosts-email-performance).
It is not something we want to forego while testing our web applications because modern email services allow painless automated email testing.
Typically email testing involves validating email fields (from, to, cc, bcc, subject, attachments), HTML content and links in the email. Email services also allow spam checks and visual checks.
The core goal is to enable the last mile of end to end testing, to enable a typical web app to be tested from start to finish.

For example imagine a scenario where a user starts having received an email invite from an organization, through company proprietary services or third party such as LinkedIn invitations.
Then, the user verifies email content, accepts the invite, and joins the organization.
Later, the user can leave the organization - or get removed by an administrator - then receives another email notification.
Using an email service, the entirety of this requirement is possible to automate and execute within seconds.

That being stated, email testing is a fundamental enabler for SaaS test architectures by permitting stateless tests that can scale; tests that independently handle their state and can be executed by _n_ number of entities at the same time.
Check out the topic [**Test States**](./sections/advanced/test-states.md) for further discussion on the topic.

<br/><br/>

## Foreword

If you are using [Gmail tricks](https://www.idownloadblog.com/2018/12/19/gmail-email-address-tricks/) or [AWS Simple Email Service](https://docs.aws.amazon.com/ses/latest/DeveloperGuide/send-email-simulator.html) and these use cases are satisfactory for your test needs without any side effects, only [topic 1](#topic-1) might be of interest to you.

There are plenty of [email testing solutions](https://www.g2.com/search/products?max=10&query=email+testing) available, and combinations of test frameworks that integrate with them.
For the code snippets and working examples, we will be using [Cypress](https://www.cypress.io/) and [Mailosaur](https://mailosaur.com/), but the ideas should generally apply to any tuple of email services and test automation frameworks.

When using Cypress with Mailosaur, there are 3 test-development approaches:

- Implement [Mailosaur API](https://docs.mailosaur.com/reference) using Cypress API testing capabilities using [`cy.request()`](https://docs.cypress.io/api/commands/request.html#Syntax) or [`cy.api()`](https://github.com/bahmutov/cy-api). Utilize plugins and helper utilities to construct test suites.

- Utilize [Mailosaur's Node package](https://www.npmjs.com/package/mailosaur) and implement them using [`cy.task()`](https://docs.cypress.io/api/commands/task.html#Syntax) which allows running node within Cypress.

- Use the [Cypress Mailosaur plugin](https://www.npmjs.com/package/cypress-mailosaur) and abstract away all the complexity!

> Check out [cypress-mailosaur-recipe](https://github.com/muratkeremozcan/cypressExamples/tree/master/cypress-mailosaur) for a working example with these approaches. Note that you will have to start a new Mailosaur trial account and replace environment variables for yourself.

<br/><br/>

## (1) Explanation & Code sample - Enabling stateless, scalable tests <a id="topic-1"></a>

Stateless tests that can scale are a necessity in any modern web application testing. We want tests that independently handle their state and tests that can be executed by _n_ number of entities at the same time.

While testing SaaS applications, which generically have Subscriptions, Users, Organizations (ex: [Slack](https://slack.com/intl/en-sk/help/articles/115004071768-What-is-Slack-#your-team-in-slack), [Cypress Dashboard Service](https://dashboard.cypress.io/organizations), etc.) a lot of the end to end workflows can rely on having unique users. Elsewise only one test execution can happen at a time and they clash with other simultaneous test executions.
This constraint reduces test automation to cron jobs or manually triggered CI.

Some ways to address unique users is by utilizing [Gmail tricks](https://www.idownloadblog.com/2018/12/19/gmail-email-address-tricks/) or [AWS Simple Email Service](https://docs.aws.amazon.com/ses/latest/DeveloperGuide/send-email-simulator.html). It is possible that you do not have to check actual email content (from, to, cc, bcc, subject, attachments etc.) and only want to have unique users, then you are on the right path with stateless tests.
However, these approaches can still be problematic; for example, non-existing emails can prompt bouncing emails to your cloud service and that can be a headache. If you want to avoid such issues and check real email content in automation, email services provide value.

Email services can also provide cost savings in test execution time by receiving emails faster, tests running quicker in the pipeline with less CI resources being consumed and less time waiting for tests to finish. If you are running 1000 pipelines a year, and save 3-4 seconds per pipeline execution, the email service can be already paying for its annual subscription just by providing extra speed.

### Achieving Stateless tests with unique emails

If in every test execution a new, unique user was used and the emails to this unique user could be verified in isolation, it would be possible to achieve a stateless test. The only side effect would be to the email service inbox, but if the test only checked emails by reference and cleaned up after itself, the email service mailbox would not be impacted.

This is easy to achieve with Mailosaur, here are two approaches to this: [Mailosaur's Node package](https://www.npmjs.com/package/mailosaur) or our own util with `faker.js`.

```javascript
// at cypress/plugins/mailosaur-tasks.js

// generates a random email address
// sample output:   ojh788.<serverId>@mailosaur.io
const createEmail = () => mailosaurClient
  .servers
  .generateEmailAddress(envVars.MAILOSAUR_SERVERID);
);

// our custom function at a helper file or commands file. The only difference is the defined prefixed name.
// sample output:  fakerJsName.<serverId>@mailosaur.io
const createMailosaurEmail = randomName =>
  `${randomName}.${Cypress.env('MAILOSAUR_SERVERID')}@mailosaur.io`;
```

<br/><br/>

## (2) Explanation - what to test in an email and how

First, let's elaborate on the setup we need.

### Test Setup and Hybrid Approach: [Mailosaur Rest API](https://docs.mailosaur.com/reference) with `cy.request()` and [Mailosaur's Node package](https://www.npmjs.com/package/mailosaur) with `cy.task()`

Mailosaur provides an [npm package](https://www.npmjs.com/package/mailosaur) and effectively all the Node code samples in the [API documentation](https://docs.mailosaur.com/reference) can be converted to `cy.task()`. Another approach is to implement Mailosaur's Rest API 'from scratch with `cy.request()`.

> Mailosaur released  [Cypress Mailosaur plugin](https://www.npmjs.com/package/cypress-mailosaur) mid 2020, and it abstracts away all the complexity with these 2 approaches. Skip to the end to see code samples and comparison.

### Environment variables

We recommend these values as environment variables. You can grab them from Mailosaur web application by creating a free trial account using any email. The trial account lasts for two weeks.

```json
  "MAILOSAUR_SERVERID": "******",
  "MAILOSAUR_PASSWORD": "******",
  "MAILOSAUR_API_KEY": "*******",
  "MAILOSAUR_API": "https://mailosaur.com/api",
  "MAILOSAUR_SERVERNAME": "user-configurable-server-name"
```

### Modularizing `cy.task()`

You can put all utilities in `cypress/plugins/index.js` file like in [this example](https://github.com/muratkeremozcan/cypressExamples/blob/master/cypress-mailosaur/cypress/plugins/index.js). A neater approach is putting all Mailosaur related tasks in its own module and importing them to the plugins file.

```javascript
// cypress/plugins/index.js

const task = require('some-plugin/task')
const percyHealthCheck = require('@percy/cypress/task') // or any other plugin you may need
const mailosaurTasks = require('./mailosaur-tasks') // our mailosaur module

// This is a pattern to merge all Cypress tasks
const all = Object.assign({}, percyHealthCheck, task, mailosaurTasks)

module.exports = (on, config) => {
  on('task', all)
}

////////

// cypress/plugins/mailosaur-tasks.js (this could be anywhere)

// the npm package
const MailosaurClient = require('mailosaur')
// we used a static file for envVars. cypress.env.json file can cause issues in CI
// There can be other solutions, do your best here.
const envVars = require('../../cypress.json')
const mailosaurClient = new MailosaurClient(envVars.MAILOSAUR_API_KEY)

// replicate Mailosaur's npm code from api docs
// https://docs.mailosaur.com/docs/fetching-messages
/** finds the most recent email message to the given email*/
const findEmailToUser = async (userEmail) => {
  let message = await mailosaurClient.messages.get(
    envVars.MAILOSAUR_SERVERID,
    {
      sentTo: userEmail,
    },
    { timeout: 25000 }
  ) // time to wait for an email to arrive
  return message
}

// other useful utilities can include the below. You can replicate them using the api docs.

// checkServerName()
// createEmail()
// deleteAMessage(messageId)
// listAllMessages()

module.exports = { checkServerName, createEmail, findEmailToUser, listAllMessages, deleteAMessage }
```

### Other Useful helper functions that Mailosaur npm package does not provide _(as far as we know)_

We can harmonize Rest API / `cy.request()` approach with npm package / `cy.task()` to build our own utility.

```javascript
/** Given user email, returns the id of the email to that user. Good example of hybrid utility functions */
const getEmailId = (email) => cy.task('findEmailToUser', email).its('id')

/** Deletes 1 email message by message id. Can be useful if you want to delete the message after running the test. */
const deleteEmailById = (id) => {
  return cy.request({
    method: 'DELETE',
    url: `${Cypress.env('MAILOSAUR_API')}/messages/${id}`,
    headers: {
      // important detail
      authorization: Cypress.env('MAILOSAUR_PASSWORD'),
    },
    auth: {
      // important detail
      user: Cypress.env('MAILOSAUR_API_KEY'),
      password: '', // any pw or empty pw will do
    },
    retryOnStatusCodeFailure: true, // because we can
  })
}

/** Deletes the most recent email sent to the user. Useful for resetting state. */
export const deleteEmail = (email) => getEmailId(email).then((id) => deleteEmailById(id))
```

<br/><br/>

## (3) Code sample - what to test in an email and how

Validating email fields (from, to, cc, bcc, subject, attachments), HTML content and links in the email.

```javascript

// an invite goes out to the recipient from the sender...

// in the cypress spec file > it block...

cy.task('findEmailToUser', recipientEmail).then(emailContent => {
  cy.wrap(emailContent).its('from')..<chain as needed>.should('eq', senderEmail); // from
  cy.wrap(emailContent).its('to')..<chain as needed>.should(..)// to
  cy.wrap(emailContent).its('cc')..<chain as needed>.should(..); // cc
  cy.wrap(emailContent).its('subject')..<chain as needed>.should(..); // subject
  // similar approach with attachments.
  // You can always end with ... .then(console.log) to take a look at the content
  // of you can check out the mailosaur email as JSON content, which makes everything easier!
  // cy.wrap(emailContent).then(console.log);

  // sample utilities to check assertions
  const html = () => cy.wrap(emailContent).its('html');
  const htmlLinks = () => html().its('links');
  const images = html().its('images');

  htmlLinks().should(..); // or chain further
  images().should(..);

  // note that you can use different styles of api assertions with Cypress
  // check out api testing examples at
  // https://github.com/cypress-io/cypress-example-recipes/tree/master/examples/blogs__e2e-api-testing
  // https://github.com/muratkeremozcan/cypressExamples/blob/master/cypress-api-testing/cypress/integration/firstTest.spec.js
});
```

## (4) The overhead is abstracted away with [Cypress Mailosaur plugin](https://www.npmjs.com/package/cypress-mailosaur)

Mailosaur team released a Cypress plugin in mid 2020. With it, we do not have to replicate any complex API utities or use cy.task via Mailosaur npm package; none of what you have seen in section (3) is necessary. There is no need to create cy.task utilities or even hyberdize them. With the Cypress Mailosaur plugin, you can just use the custom Cypress commands Mailosaur team created for us.

All we need is to install the package `npm install cypress-mailosaur --save-dev` and add the following line to cypress/support/index.js:
`import 'cypress-mailosaur'`.

Mailsaur plugin has a few handy functions which help you abtract complex needs.
A full list can be found at at https://github.com/mailosaur/cypress-mailosaur

Here is the plugin version of the above code. The usage is somewhat similar, but we did not have implement any cy.task() utilities, custom helper functions or hybrid helpers. We also get new, easy to use helper functions that work seamlessly.

You can find a working version of this code and the above at the [link](https://github.com/muratkeremozcan/cypressExamples/tree/master/cypress-mailosaur).
```javascript
it('uses the plugin to check the email content (no need for creating complex utilities with cy.task) ', function () {
    const userEmail = createEmail(internet.userName());
    cy.task('sendSimpleEmail', userEmail); // an npm package to send emails, usually your app would do this

    // a convenient helper functions to list mesages
    cy.mailosaurListMessages(Cypress.env('MAILOSAUR_SERVERID')).its('items').its('length').should('not.eq', 0);

    // this helper command replaces the complex cy.task('findEmailToUser') utility we had to create
    cy.mailosaurGetMessage(
      Cypress.env('MAILOSAUR_SERVERID'),
      { sentTo: userEmail },
      // note from Jon at Mailosaur:
      // The get method looks for messages received within the last hour
      // if looking for emails existing before that, you have to add this. Optional otherwise
      // { receivedAfter: new Date('2000-01-01') }
    ).then(emailContent => {
      // this part is the same
      cy.wrap(emailContent).its('from').its(0).its('email').should('contain', 'test@nodesendmail.com');
      cy.wrap(emailContent).its('to').its(0).its('email').should('eq', userEmail);
      cy.wrap(emailContent).its('subject').should('contain', 'MailComposer sendmail');
    });

    // alternate approach to getting message by sent to'
    cy.mailosaurGetMessagesBySentTo(Cypress.env('MAILOSAUR_SERVERID'), userEmail).then(emailItem => {
      // the response is slightly different, but you can modify it to serve the same purpose
      const emailContent = emailItem.items[0];
      cy.wrap(emailContent).its('from').its(0).its('email').should('contain', 'test@nodesendmail.com');
      cy.wrap(emailContent).its('to').its(0).its('email').should('eq', userEmail);
      cy.wrap(emailContent).its('subject').should('contain', 'MailComposer sendmail');
    });

    // an easy to use bonus utility for checking spam score
    cy.mailosaurGetMessagesBySentTo(Cypress.env('MAILOSAUR_SERVERID'), userEmail).its('items').its(0).its('id').then(messageId => {
      // does convenient spam analysis
      cy.mailosaurGetSpamAnalysis(messageId).its('score').should('eq', 0);
      // you can observe the console output with a plain "cy.mailosaurGetSpamAnalysis(messageId);  " and check for deeper assertions
    })
  });


```