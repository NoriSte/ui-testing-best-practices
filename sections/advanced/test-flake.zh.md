# 不稳定的测试

<br/><br/>

## 一段简要说明

每次测试都必须产生一致的结果，而可重复的流水线执行结果则是至关重要的。如果测试无法产生可靠的结果，将降低对测试的信心，还需要进行维护，这将降低所有价值。在这些情况下，最好进行手动功能测试。

并请自问以下几个问题：

- 如何解决测试波动，通过成长的过程确保测试的可信度？
- 如何处理流水线、基础设施、共享资源等方面的假阴性，并在没有控制的情况下解决？
- 如何发现零星缺陷？

<br/><br/>

## 第一步：本地识别不稳定的测试

推荐在模拟流水线 CI 机器的操作系统中进行无头模式执行；Linux 和 MacOS 与流水线的行为更为相似，而 Windows 则是个例外，除非你正在使用 Windows Docker 容器。无头执行将更容易暴露测试波动。有多种方法可以重复执行测试规范，Cypress 提供的一个例子是使用 Lodash 库（Cypress 已经内置了）`Cypress._.times( <重复次数>, () => { <你的测试规范代码> })`。在提交代码合并请求之前，务必使用此方法。

### 第一步的代码示例

```JavaScript
// will repeat the full suite 10 times
Cypress._.times( 10, function {

  describe('..', function () {

    before(function () {
    });

    beforeEach(function () {
    });

      // you can place it anywhere to repeat 1 test, or another describe / context block
      Cypress._.times( 3, function {
        it('..', function () {..});
      }
      it('..', function () {..});
      it('..', function () {..});
      it('..', function () {..});

  });
});

// this will result in 6 tests per run x 10 runs = 60 executions

```

<br/>

## 第二步：在流水线中识别不稳定的测试并进行重试

在初始的流水线顺利通过并合并代码后，**有时**测试会出现失败的情况。

为什么测试在**没有可重现的缺陷**且**测试代码已经完全优化的情况下仍然失败呢**？

为了解决这种零星的失败问题，以及避免测试被忽略或**降低团队对其的信心**，我们可以采用重试机制：

- 用以解决团队无法掌控的不可靠流水线基础设施问题
- 在开发中遇到的问题，或者依赖于正在开发中的外部服务
- 最为重要的是，**用于锁定零星的系统问题**

### 第二步的代码示例

许多框架都提供了重试实用工具。下面是一个例子来自于 [Cypress 文档](https://docs.cypress.io/guides/references/migration-guide.html#Tests-retries):

在一个测试中：

```javascript
it('allows user to login', { // can also be in a context or describe block
  retries: {
    runMode: 2, // for CI usage
    openMode: 1  // for local usage
  }
}, () => {
  // ...
})
```

在配置文件中，例如 `cypress.json`:

```json
{
  "retries": {
    "runMode": 1,
    "openMode": 3
  }
}



```

## 第三步：识别零星的系统问题 - *不稳定的系统*

鉴于以下情况：

- 不存在可重现的缺陷
- 测试代码已经充分优化
- 已知并通过测试重试有效解决了流水线问题
- 已知、认可并通过测试重试解决了外部依赖和成长痛苦

... 我们如何检测系统存在的更深层次问题，这可能表明存在*不稳定的系统*？以下是团队[Cypress 仪表板](https://www.cypress.io/dashboard/)上的一个示例快照：

>*“在周末的 40 次执行中，它以 10% 的错误率失败... 我们运行了测试套件 40 次，在其中的一次执行中看到该规范重试了 2 次，直到通过...”*

*请注意：相机图标表示一些测试失败，因为 Cypress 在失败时会拍摄视频和截图。*

在这些情况下，可以通过每晚或周末的 [cron 任务](https://crontab.guru/#0_1-23_*_*_6-7) 进行一致性测试，作为更深层次系统问题的初始指标。这些通常是那些容易泄漏到生产环境中、在实际使用中被发现并具有昂贵后果的模糊缺陷。

### 代码示例 - [cron 任务](https://crontab.guru/#0_1-23_*_*_6-7)

```cron syntax
at minute 0 at midnight and 2 am, every day-of-week from Monday through Friday:

0 0,2 * * 1-5


At minute 0 past hour 2, 6, 8, 10, 12, 14, 16, 18, and 20 on every day-of-week from Saturday through Sunday:

0 2,6,8,10,12,14,16,18,20 * * 6-7
```

一旦排除了所有其他因素，并且在管道中使用 cron 任务自动化测试初步指示了“系统波动”，这些问题就是**性能测试**的理想候选项，因为这种测试方法可以直接指出可能导致“不稳定的系统”的系统缺陷。

性能测试的要点如下：
![ ](../../assets/images/performanceTesting.jpg)

有许多性能测试工具，其中一个我们认为比较易于使用的是 [k6-loadImpact](https://docs.k6.io/docs)，因为它采用了 ES6 语法，并且与流水线兼容。
你可以在 [这里](https://github.com/muratkeremozcan/k6-loadImpact) 找到一个包含代码示例的简单教程。

## 参考资料

[Google 测试博客：我们的测试中哪些是不稳定的，是从哪些方面产生的](https://testing.googleblog.com/2017/04/where-do-our-flaky-tests-come-from.html)
