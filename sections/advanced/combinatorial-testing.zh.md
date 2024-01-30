# 组合测试

<br/><br/>

## 一段简要说明

* [组合测试](http://csrc.nist.gov/Projects/automated-combinatorial-testing-for-software) 是一种经过验证的、成本较低的、更为有效的软件测试方法。
* 这种测试的关键思想是，并非每个参数都对每次故障都有影响，而是大多数故障是由相对较少的参数之间的相互作用引起的。
* 与传统方法相比，测试参数组合可以更有效地检测故障。

美国国家标准与技术研究院[NIST](https://www.nist.gov/) 在 1999 年到 2004 年进行的一系列研究表明，大多数软件缺陷和故障是由一个或两个参数引起的，逐渐减少到由三个或更多参数引起的。这一发现被称为“交互规则”，对软件测试具有重要意义，因为这意味着测试参数组合可以比传统方法更有效地检测故障。NIST 和其他机构收集的数据表明，软件故障仅由少数几个变量的相互作用引发（不超过六个）。有时使用成对（2 路组合）测试可以以较低的成本获得相当不错的结果，通常不低于 60% 的故障覆盖率，但这可能对于关键任务的软件来说可能不足够。

<br/><br/>

## (1) 代码示例 – 产品负责人问题

一位产品负责人曾提出一个问题：
> "从最佳实践或实际角度来看，你是否应该在每种可能的配置下测试系统？
例如，假设你有 A、B、C、D、E 五个功能，客户 1 拥有 A/B，客户 2 拥有 A/B/C，客户 3 拥有 A/D，客户 4 拥有 B/D，客户 5 拥有 A/B/C/D/E....
你是否应该测试每种可能的功能组合，还是测试每个功能单独，如果它们在独立测试中能够正常工作，就相信它们整体上也能正常工作？"

5 个客户和 5 个功能，详尽无遗将需要 25 个测试。
在描述的约束条件下，只需要 14 个测试。
为了提供一个代码示例，我们将使用描述规格的[CTWedge](https://foselab.unibg.it/ctwedge/)脚本化组合模型。还有许多其他列在[pairwise.org](http://pairwise.org/)上的 CT 工具。我们（在西门子）使用过的其他一些工具包括[ACTs](https://csrc.nist.gov/projects/automated-combinatorial-testing-for-software)和[CAgen](https://matris.sba-research.org/tools/cagen/#/workspaces)。

```
Model POquestion
 Parameters:
   features : {A, B, C, D, E}
   customer:  {1, 2, 3, 4, 5}

 Constraints:
   # customer = 1 => features = A || features = B #
   # customer = 2 => features = A || features = B || features = C #
   # customer = 3 => features = A || features = D #
   # customer = 4 => features = B || features = D #
   # customer = 5 => features = A || features = B || features = C || features = D || features = E #
```

在这里粘贴脚本以生成结果 [这里](http://foselab.unibg.it/ctwedge/)。

测试的目标是检验参数之间的双向（或更多）相互作用。当只有两个参数时，收益并不太明显，因为这是一种穷举的方法。

如果参数数量超过两个，对它们之间的双向交互进行覆盖将确保找到该领域可能存在的 60-99% 的所有潜在缺陷。三向交互为 90%，四向为 95%，五向为 97%，六向为 100%。

![组合测试图](../../assets/images/combinatorial-testing/ct-graph.PNG)

在这个例子中，通过添加另一个*参数*，我们称之为 `configuration`，并假设有 5 种可能的配置 / *参数值*。这将生成一个包含 125 个测试的详尽套件。

```Text
Model POquestion
 Parameters:
   features : {A, B, C, D, E}
   customer:  {1, 2, 3, 4, 5}
   configuration: {config1, config2, config3, config4, config5}

 Constraints:
   # customer = 1 => features = A || features = B #
   # customer = 2 => features = A || features = B || features = C #
   # customer = 3 => features = A || features = D #
   # customer = 4 => features = B || features = D #
   # customer = 5 => features = A || features = B || features = C || features = D || features = E #
```

将其粘贴到 [CTWedge](https://foselab.unibg.it/ctwedge/) 上，这将生成一个包含 31 个测试的测试套件。如果添加一些约束，表明某些特性不应该与某些配置一起工作，甚至可以进一步精简。

请注意，组合测试的建模可以并且确实包含等价分区、边界值分析和其他技术。模型越准确，测试套件的故障检测能力就越强。

<br/><br/>

## (2) 代码示例 – NASA 的开关板共有 34 个开关

以 NASA 的一个例子为参考，有 34 个开关，每个开关可以处于打开或关闭的状态。要进行详尽的测试，有 170 亿种可能的组合方式。

![ ](../../assets/images/combinatorial-testing/nasa-switches.PNG)

不必测试所有的 2^34 种可能性。通过使用组合测试进行建模，你可以根据风险做出经过计算的决策。

```text
Model NASAswitches

Parameters:
    switch1: Boolean
    switch2: Boolean
    switch3: Boolean
    switch4: Boolean
    switch5: Boolean
    switch6: Boolean
    switch7: Boolean
    switch8: Boolean
    switch9: Boolean
    switch10: Boolean
    switch11: Boolean
    switch12: Boolean
    switch13: Boolean
    switch14: Boolean
    switch15: Boolean
    switch16: Boolean
    switch17: Boolean
    switch18: Boolean
    switch19: Boolean
    switch20: Boolean
    switch21: Boolean
    switch22: Boolean
    switch23: Boolean
    switch24: Boolean
    switch25: Boolean
    switch26: Boolean
    switch27: Boolean
    switch28: Boolean
    switch29: Boolean
    switch30: Boolean
    switch31: Boolean
    switch32: Boolean
    switch33: Boolean
    switch34: Boolean
```

在 [CTWedge](https://foselab.unibg.it/ctwedge/) 中通过下拉菜单开关测试的相互作用次数。

* 14 次测试：通过开关之间的 2 次相互作用引起的故障 - 可根据产品找到 60-99% 的所有潜在故障
* 33 次测试：通过开关之间的 3 次相互作用引起的故障 - 可根据产品找到 90-99% 的所有潜在故障
* 85 次测试：通过开关之间的 4 次相互作用引起的故障 - 可根据产品找到 95-99% 的所有潜在故障
* 220 次测试：通过开关之间的 5 次相互作用引起的故障 - 可找到超过 99% 的所有潜在故障
* 538 次测试：通过开关之间的 6 次相互作用引起的故障 - 可找到所有潜在故障的 100%

<br/><br/>

## (2) 代码示例 - [西门子楼宇操作员 CI 配置](https://cypress.slides.com/cypress-io/siemens-case-study#/16)

参考上面的幻灯片链接或[直播视频](https://www.youtube.com/watch?v=aMPkaLOpyns&t=1624s)以获取有关如何使用[CAMetrics](https://matris.sba-research.org/tools/cametrics/#/new)测量组合覆盖率的详细说明。基本上，你可以使用任何组合测试工具生成一个 CSV 文件，然后将其拖放到 CAMetrics 中。之后，CAMetrics 可以为你提供各种组合覆盖率报告。

> 请注意，将 [CSV 转换为 JSON](https://www.csvjson.com/csv2json) 非常简单，然后可以使用 JSON 文件在所选的任何测试框架中进行数据驱动测试。

```text
Model CI
 Parameters:
   deployment_UI : { branch, development, staging }
   deployment_API:  { development, staging }
   spec_suite: { ui_services_stubbed, ui_services, ui_services_hardware, spot_check}
   browser: { chrome, electron, firefox }

 Constraints:
   // one extra constraint for firefox spot checks
   # browser=firefox <=> spec_suite=spot_check #
   // on staging, run all tests
   # spec_suite=ui_services_hardware <=> deployment_API=staging #
   // match dev vs dev, staging vs staging, and when on staging use Chrome
   # deployment_UI=development => deployment_API=development #
   # deployment_UI=staging => deployment_API=staging #
   # deployment_UI=staging && deployment_API=staging => browser=chrome #
   // when on branch, stub the services
   # deployment_UI=branch => spec_suite=ui_services_stubbed #
   // do not stub the services when on UI development
   # deployment_UI=development => spec_suite!=ui_services_stubbed #
```

## 参考资料和延伸阅读

[自动化组合测试软件](https://csrc.nist.gov/Projects/automated-combinatorial-testing-for-software)

[幻灯片 16-50：探讨自动化和组合纪律在辅助探索性测试方面的应用](https://prezi.com/tpffqit1yn87/utilization-of-automation-and-combinatorial-disciplines-in-aid-of-exploratory-testing/)

[西门子工业公司建筑技术部实际组合测试方法的应用](https://ieeexplore.ieee.org/document/7899057?section=abstract)

[现代 Web 开发中组合测试的工业研究](https://ieeexplore.ieee.org/document/8728910)

[在大型组织中引入组合测试](https://ieeexplore.ieee.org/document/7085645/)

[组合策略的输入参数建模](http://barbie.uta.edu/~mehra/1%20INPUT%20PARAMETER%20MODELING%20FOR%20COMBINATION%20STRATEGIES.pdf)

[组合模型中的常见模式](http://barbie.uta.edu/~mehra/62_Common%20Patterns%20in%20Combinatorial%20Models.pdf)

[等效类和两层覆盖阵的高效验证和同时测试](https://tsapps.nist.gov/publication/get_pdf.cfm?pub_id=917899)
