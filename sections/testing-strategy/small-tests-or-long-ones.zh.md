
# 单个长的端到端测试还是多个小的独立测试？

<br/><br/>

## 一段简要说明

在讨论对 CRUD 应用进行测试时，我们应该如何组织“创建”、“修改”和“删除”端到端（E2E）测试呢？

完整的选项列表如下：

1. **有三个小的 E2E 测试，依赖于执行顺序**（测试 B 假设测试 A 已运行）- 这是唯一的不良解决方案，我将解释原因。
2. **有三个小的 E2E 测试，独立于执行顺序**（测试 B 不受测试 A 是否运行的影响）- 从理论上讲，是最好的解决方案。但仍然需要大量样板代码，而且为了快速执行。
3. **有一个执行所有操作的扩展 E2E 测试** - 对于本文介绍的案例来说，这是一个很好的折中方案。

这取决于情况，我提到的大多数问题与 E2E 测试的隐含问题有关，这是我们应该尽量减少这类测试的强烈信号。作为前端工程师，我更喜欢投资时间编写无需服务器的测试，而不是 E2E 测试。继续阅读，你将了解原因。

<br/><br/>

## 1 - 有三个小的 E2E 测试，依赖于执行顺序（测试 B 假设测试 A 已运行）

测试流程如下：

1. 开始（*应用程序状态为空*）
2. 测试 1: 创建实体
3. 测试 2: 修改实体
4. 测试 3: 删除实体
5. 结束（*应用程序状态为空*）

在这种情况下，这些测试不是独立的，而是依赖于执行顺序。为了测试 CRUD 流程，有三个主要测试："创建实体"、"修改实体"、"删除实体"。第二个测试（"修改实体"）假设在其启动时应用程序状态是正确的，因为它在 "创建实体" 之后运行。"删除实体" 也必须在 "修改实体" 之后运行，依此类推。

将多个测试耦合在一起是一种反模式，原因如下：

- **误报**：一旦一个测试失败，后续测试会连续失败。
- **难以调试**：由于不确定性较高，理解失败的根本原因更加复杂。测试失败是因为代码本身失败？还是因为先前测试的状态发生了变化？然后，当一个测试失败时，你必须调试两个测试。
- **难以调试**（再次）：开发人员会浪费大量时间，因为他们无法运行单个测试，也无法使用 `skip` 和 `only` 仅运行其中一部分测试。
- **难以重构**：测试无法移动到其他位置。如果测试代码变得太长、太复杂等，你无法将其移动到专用文件/目录中，因为它依赖于先前的测试。
- **难以阅读**：读者无法知道一个测试的作用，因为他们还必须了解先前的测试。你必须阅读两个测试，而不是一个，这是不好的。

我不建议以这种方式编写耦合的测试，但我想包含它们以确保你明白原因。

## 2 - 设计三个小型端到端（E2E）测试，使其独立于执行顺序

为了确保每个测试的独立性，每个测试在运行前都应该创建所需的应用程序状态，然后在完成后进行清理。相较于原有的顺序（创建->修改->删除），前文提到的流程应该调整如下（*斜体* 表示与原有流程相比的新步骤）：

1. **开始**（*应用程序状态为空*）
2. **测试 1：创建实体**
   1. ***之前***：加载页面（*应用程序状态为空*）
   2. 创建实体
   3. ***之后***：删除*实体*（*应用程序状态为空*）
3. **测试 2：修改实体**
   1. ***之前***：通过 API 创建*实体*
   2. ***之前***：加载页面（*应用程序状态为空*）
   3. 修改实体
   4. ***之后***：通过 API 删除*实体*（*应用程序状态为空*）
4. **测试 3：删除实体**
   1. ***之前***：通过 API 创建*实体*
   2. ***之前***：加载页面（*应用程序状态为空*）
   3. 删除实体
   4. ***之后***：删除操作（*应用程序状态为空*）
5. **结束**（*应用程序状态为空*）

通过这种方式，每个测试都是相互独立的。需要注意的是，之前和之后的操作直接通过调用服务器 API 完成，因为通过 UI 完成这些操作将会很慢。然而，这种方法的问题在于**测试变得更加耗时**，因为每个测试都需要创建实体，并且每个测试都需要访问页面。当应用程序加载需要花费 10 秒钟时（Hasura 的控制台最初的情况），重新加载应用程序将成为一个问题。

为了确保测试既独立又高效，我们需要进一步改进上述流程：

- 充分利用前一个测试的应用状态。
- 同时，如果尚未运行测试，还需要创建所需的应用状态。

具体来说，流程如下（与前一章节相比，*斜体*表示新步骤）：

1. **开始**（*应用状态为空*）
2. **测试 1：** 创建实体
    1. ***之前***：*实体* 是否存在？
        1. *否：没问题！*
        2. *是：通过 API 删除实体*
    2. ***之前***：加载页面（*应用状态为空*）
    3. 创建实体
3. **测试 2：** 修改实体
    1. ***之前***：*实体* 是否存在？
        1. *是：没问题！*
        2. *否：通过 API 创建实体*
    2. ***之前***：*实体* 是否已包含测试即将进行的更改？
        1. *是：没问题！*
        2. *否：通过 API 修改实体*
    3. ***之前***：我们是否已经在正确的页面上？
        1. *是：没问题！*
        2. *否：加载页面*
    4. 修改实体
4. **测试 3：** 删除实体
    1. ***之前***：实体是否存在？
        1. *是：没问题！*
        2. *否：通过 API 创建实体*
    2. ***之前***：我们是否已经在正确的页面上？
        1. *是：没问题！*
        2. *否：加载页面*

5. 删除实体
6. **结束**（*应用状态为空*）

现在，如果你一次运行所有测试，每个测试都会利用之前测试的应用状态。如果只运行“修改实体”测试，它会创建所需的一切，然后运行测试本身。

现在我们既有测试的独立性又有测试的性能！很不错！

嗯... 你是否注意到我们需要编写大量代码？[cypress-data-session](https://github.com/bahmutov/cypress-data-session) 插件很方便，但存在两个问题：

1. 有很多与 cypress-data-session 相关的样板代码
2. 在 E2E 测试中，必须维护许多可能与主应用程序中使用的 API 调用不同步的 API 调用。

这是一个与 cypress-data-session 相关的样板代码示例（来自 Hasura Console 代码库）。

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

以下是用于创建实体的 API 调用示例（来自 Hasura Console 代码库）。

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

因此，拥有独立的测试是至关重要的，但也伴随着一些成本。

这就是为什么，针对这个具体问题，我选择了最后一种选择...

## 3 - 进行一次全面的端到端测试

优点：可以减少很多样板文件。

缺点：与测试一起工作变得更慢了（你不能再仅运行第三个测试了）

与我们需要编写的样板和需要维护的代码相比，将它们统一起来是值得的。毕竟，我正在处理的特定 CRUD 流程大约需要 20 秒。

1. 开始 (*应用程序状态为空*)
2. 测试：CRUD
   1. ***之前****：如果存在实体，则删除它（应用程序状态为空）*
   2. ***之前****：加载页面*
   3. 创建实体
   4. 修改实体
   5. 删除实体
   6. ***之后****：如果存在实体，则删除它（应用程序状态为空）*
3. 结束 (*应用程序状态为空*)

同时，这也使得 cypress-data-session 变得无用。因此，少了一个需要保持更新的依赖。

## 结论

处理端到端测试很困难。处理真实数据、清除真实应用程序状态等都是有成本的。我知道端到端测试是唯一能够提供完整信心的测试，但作为一名前端工程师（请记住，我不是 QA 工程师），我更愿意使用无需服务器的测试。

## 相关章节

- 🔗 [从金字塔的顶端着手构建测试！](/sections/beginners/top-to-bottom-approach.zh.md)
- 🔗 [把你的测试工具当作主要的开发工具来使用](/sections/generic-best-practices/use-your-testing-tool-as-your-primary-development-tool.zh.md)
<br/><br/>

*由 [NoriSte](https://github.com/NoriSte) 在 [dev.to](https://dev.to/noriste/decouple-the-back-end-and-front-end-test-through-contract-testing-112k)上进行了联合发表。*
