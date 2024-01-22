# 邮件测试

<br/><br/>

## 一段简要说明

电子邮件测试对于[业务成功至关重要](https://www.industrialmarketer.com/why-email-testing-is-critical-for-email-marketing-success/)，[能够提升电子邮件的表现](https://litmus.com/blog/3-reasons-why-email-testing-boosts-email-performance)。在测试 Web 应用时，我们绝不希望忽略这一点，因为现代电子邮件服务使得自动化电子邮件测试变得轻而易举。通常，电子邮件测试包括验证电子邮件字段（发件人、收件人、抄送、密送、主题、附件）、HTML 内容以及电子邮件中的链接。电子邮件服务还支持垃圾邮件检查和视觉检查。

其核心目标在于实现端到端测试的最后一步，确保典型的 Web 应用能够从头到尾得到全面测试。

以一个场景为例，用户收到来自组织的电子邮件邀请，可以是通过公司专有服务或第三方平台，比如 LinkedIn 邀请。接着，用户验证电子邮件内容，接受邀请，并加入该组织。随后，用户可以选择离开组织，或者被管理员移除，然后再次收到另一封电子邮件通知。通过电子邮件服务，这一需求的整个过程可以在短短几秒钟内自动完成和执行。

值得注意的是，电子邮件测试是 SaaS 测试架构的基础，通过允许无状态测试来实现可伸缩性，这些测试能够独立处理其状态，并能够同时由多个实体执行。详细讨论请参阅 [**测试状态**](./sections/advanced/test-states.zh.md)。

<br/><br/>

## 前言

如果你正在使用[Gmail 技巧](https://www.idownloadblog.com/2018/12/19/gmail-email-address-tricks/)或[AWS Simple Email Service](https://docs.aws.amazon.com/ses/latest/DeveloperGuide/send-email-simulator.html)，并且这些用例在没有任何副作用的情况下能够满足你的测试需求，那么可能只有[topic 1](#topic-1)对你感兴趣。

市面上有很多[电子邮件测试解决方案](https://www.g2.com/search/products?max=10&query=email+testing)，以及与它们集成的测试框架的组合。在代码片段和实际示例中，我们将使用[Cypress](https://www.cypress.io/)和[Mailosaur](https://mailosaur.com/)，但这些思想通常适用于任何电子邮件服务和测试自动化框架的组合。

在使用 Cypress 与 Mailosaur 时，有三种测试开发方法：

- 利用 Cypress API 测试功能实现[Mailosaur API](https://docs.mailosaur.com/reference)，使用[`cy.request()`](https://docs.cypress.io/api/commands/request.html#Syntax)或[`cy.api()`](https://github.com/bahmutov/cy-api)。利用插件和辅助工具构建测试套件。

- 利用[Mailosaur 的 Node 包](https://www.npmjs.com/package/mailosaur)并使用[`cy.task()`](https://docs.cypress.io/api/commands/task.html#Syntax)实现，该方法允许在 Cypress 内运行 Node。

- 使用[Cypress Mailosaur 插件](https://www.npmjs.com/package/cypress-mailosaur)并将所有复杂性抽象掉！

> 请查看[cypress-mailosaur-recipe](https://github.com/muratkeremozcan/cypressExamples/tree/master/cypress-mailosaur)以获取这些方法的实际示例。请注意，你将需要启动一个新的 Mailosaur 试用账户并替换自己的环境变量。

<br/><br/>

## (1) 解释 & 代码示例 - 启用无状态、可伸缩的测试

在任何现代 Web 应用测试中，能够实现可伸缩的无状态测试是至关重要的。我们追求的是那些能够独立处理其状态，并能够同时由 n 个实体执行的测试。

在测试 SaaS 应用程序时，通常会涉及订阅、用户、组织等通用概念（例如[Slack](https://slack.com/intl/en-sk/help/articles/115004071768-What-is-Slack-#your-team-in-slack)，[Cypress Dashboard Service](https://dashboard.cypress.io/organizations)等），很多端到端工作流可能依赖于拥有唯一用户。否则，一次只能执行一个测试，并且可能与其他同时进行的测试执行发生冲突。这种约束将测试自动化限制在定时作业或手动触发的 CI 中。

解决唯一用户问题的一些方法包括利用[Gmail 技巧](https://www.idownloadblog.com/2018/12/19/gmail-email-address-tricks/)或[AWS Simple Email Service](https://docs.aws.amazon.com/ses/latest/DeveloperGuide/send-email-simulator.html)。如果你只想要唯一的用户而不必检查实际的电子邮件内容（发件人、收件人、抄送、密送、主题、附件等），那么使用无状态测试是正确的路径。

然而，这些方法仍然可能存在问题；例如，不存在的电子邮件可能导致回送邮件到你的云服务，这可能会让人头疼不已。如果你想要避免这些问题并在自动化中检查实际的电子邮件内容，电子邮件服务提供了有价值的功能。

电子邮件服务还可以通过更快地接收电子邮件，在流水线中更快地运行测试，消耗更少的 CI 资源以及减少等待测试完成的时间，为测试执行时间提供成本节省。例如，如果你每年运行 1000 个流水线，并每个流水线执行节省 3-4 秒，电子邮件服务可能已经通过提供额外的速度来支付其年度订阅费用。

### 实现具有唯一电子邮件的无状态测试

如果每次测试执行都使用一个新的、唯一的用户，并且可以单独验证发送给这个唯一用户的电子邮件，那么就可以实现无状态测试。唯一的副作用将仅影响电子邮件服务的收件箱，但如果测试只通过引用检查电子邮件并在测试结束后进行清理，电子邮件服务的邮箱将不受影响。

通过 Mailosaur 实现这一点非常容易，以下是两种方法：[Mailosaur 的 Node 包](https://www.npmjs.com/package/mailosaur)或使用`faker.js`创建我们自己的工具。

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

## (2) 解释 - 在电子邮件中进行何种测试以及如何进行测试

首先，让我们详细说明我们需要的设置。

### 测试设置和混合方法

[Mailosaur Rest API](https://docs.mailosaur.com/reference) 使用 `cy.request()` 和 [Mailosaur 的 Node 包](https://www.npmjs.com/package/mailosaur) 使用 `cy.task()`

Mailosaur 提供了一个[npm 包](https://www.npmjs.com/package/mailosaur)，实际上[API 文档](https://docs.mailosaur.com/reference)中的所有 Node 代码示例都可以转换为`cy.task()`。另一种方法是使用`cy.request()`从零开始实现 Mailosaur 的 Rest API。

> Mailosaur 在 2020 年中发布了[Cypress Mailosaur 插件](https://www.npmjs.com/package/cypress-mailosaur)，它通过这两种方法抽象出所有复杂性。请跳到最后查看代码示例和比较。

### 环境变量

我们建议将以下值设置为环境变量。你可以通过使用任何电子邮件地址创建一个免费试用帐户，并从 Mailosaur Web 应用程序中获取这些值。试用帐户的有效期为两周。

```json
  "MAILOSAUR_SERVERID": "******",
  "MAILOSAUR_PASSWORD": "******",
  "MAILOSAUR_API_KEY": "*******",
  "MAILOSAUR_API": "https://mailosaur.com/api",
  "MAILOSAUR_SERVERNAME": "user-configurable-server-name"
```

### 模块化 `cy.task()`

你可以将所有实用工具放在`cypress/plugins/index.js`文件中，就像在[此示例](https://github.com/muratkeremozcan/cypressExamples/blob/master/cypress-mailosaur/cypress/plugins/index.js)中一样。更整洁的方法是将所有与 Mailosaur 相关的任务放在其自己的模块中，并将它们导入到插件文件中。

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

### 其他有用的辅助函数，Mailosaur npm 包目前似乎不提供（据我们所知）

我们可以将 Rest API / `cy.request()`方法与 npm 包 / `cy.task()`方法协调一致，以构建我们自己的实用程序。

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

## (3) 代码示例 - 在电子邮件中进行何种测试以及如何进行测试

验证电子邮件字段（发件人、收件人、抄送、密送、主题、附件）、电子邮件中的 HTML 内容和链接。

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

## (4) 这个开销被[Cypress Mailosaur 插件](https://www.npmjs.com/package/cypress-mailosaur)抽象掉了

Mailosaur 团队在 2020 年中发布了一个 Cypress 插件。通过使用它，我们无需复制任何复杂的 API 工具，也无需使用 Mailosaur npm 包通过 cy.task 进行操作；在第（3）节中看到的内容都是不必要的。没有必要创建 cy.task 实用程序，甚至不需要混合它们。使用 Cypress Mailosaur 插件，你只需使用 Mailosaur 团队为我们创建的自定义 Cypress 命令。

我们只需安装`npm install cypress-mailosaur --save-dev`并在 cypress/support/index.js 中添加以下行：
`import 'cypress-mailosaur'`。

Mailosaur 插件有一些方便的函数，可以帮助你抽象出复杂的需求。
完整列表可以在 <https://github.com/mailosaur/cypress-mailosaur> 上找到

以下是上述代码的插件版本。使用方式有些类似，但我们无需实现任何 cy.task() 实用程序、自定义帮助函数或混合帮助程序。我们还获得了新的、易于使用的辅助函数，可以无缝运行。

你可以在[链接](https://github.com/muratkeremozcan/cypressExamples/tree/master/cypress-mailosaur)中找到这个代码和上面的工作版本。

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
