# 测试状态

<br/><br/>

## 一段简要说明

测试应该是可重复的、模块化的，并且应该自己处理状态设置。为了为其他测试实现状态，不应该重复执行 UI 测试。

我们希望测试是无状态的，具有可扩展性：

- 测试应该独立处理其状态。
- 没有对外部产生不受控制的副作用，或者具有测试自身能够处理的可管理副作用。
- 测试应该能够被 *n* 个实体同时执行。

<br/><br/>

## 代码示例 – 解释说明

**可重复性**: 测试必须能够设置状态、执行测试，并在不影响下一个测试执行的前提下使环境保持干净。如果一个测试在每次执行时都使系统混乱，将其留在无法重置的状态，那么这个测试就适合作为手动测试。测试还不能互相冲突：多个测试者和流水线必须能够同时执行相同的测试。如果这不可行，这些测试组应该每天在流水线中执行一次，最好在非工作时间执行 [cron 作业](https://crontab.guru/#0_1-23_*_*_6-7)。

每个需要更改环境状态的测试都必须被用作设置 - 状态 - 测试，并确保在下一个测试之前能够清理测试环境。

最好是 UI 测试不要重复作为设置测试；在必须将 UI 测试用作另一个测试的设置的情况下，应该使用 API 测试、应用程序操作或数据库初始化。

**设置 vs 清理**: 设置（之前全部）优于清理（之后全部）。在可能的情况下，测试本身应该负责在一个干净的环境中开始。然而，正如上面强调的，测试不能使得在它们执行后下一个测试无法清理环境。

**登录**: UI 登录的各种形式应仅在其各自的测试用例中使用。任何其他需要登录的测试应该使用内部的 API 登录和/或具有预配置的测试用户。

**测试状态设置**: 鼓励测试是隔离的，以便它们在执行之前不依赖于整个设置。例如：如果一组测试可能需要创建用户，可以利用一个测试用户在隔离中使用这些测试。另一方面，设置用户的测试应该是独立的和隔离的。

**模块化**: 每个测试应该能够独立运行，不依赖于其他测试来为其设置状态。如果需要进行这样的设置，应该在 `beforeAll` 或 `beforeEach` 部分进行。测试这一点的一个好方法是在隔离中运行测试：`it.only()`，`fit()`，等等。

```JavaScript
describe('..', function () {

  // setup (before/beforeEach) is preferred over cleanup (after/afterEach)

  before(function () {
    // login with UI once in an isolated test
    // for login here and all other tests, use a faster login method: use API, App Actions or DB seeding
  });

  beforeEach(function () {
    // setup additional state...
    // have one UI test to ensure this state can be achieved
    // however for the state set up here, utilize API, Application Actions or DB seeding; do not repeat UI tests
  });

    // test each test once with .only to ensure modularity
    it('..', function () {..});
    it('..', function () {..});
    it.only('..', function () {..});
    it('..', function () {..});

});

```

<br/><br/>

## 参考资料

[放弃使用页面对象，转而使用应用动作](https://www.cypress.io/blog/2019/01/03/stop-using-page-objects-and-start-using-app-actions/)

[Cypress 文档：测试组织、登录、状态控制](https://docs.cypress.io/guides/references/best-practices.html#Organizing-Tests-Logging-In-Controlling-State)
