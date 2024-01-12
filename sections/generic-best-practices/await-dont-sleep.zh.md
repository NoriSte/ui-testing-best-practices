# 等待，不要休眠

<br/><br/>

## 一段简要说明

在测试 UI 时，您需要定义应用程序必须经过的关键点。到达这些关键点是一个异步过程，因为几乎 100% 的情况下，UI 不会同步更新。

这些关键点称为**确定性事件**，即您知道必须发生的事件。

具体取决于您定义的事件以及 UI 达到这些事件的方式，但通常会存在一些“长时间”等待，例如 XHR 请求，以及一些更快的等待，例如重新渲染更新。

解决异步更新的方法似乎很简单：**休眠/暂停测试**一段时间，几毫秒、几分之一秒，甚至几秒钟。这可以使测试正常工作，因为它给应用程序足够的时间来更新自身并移动到下一个要测试的确定性事件。

请注意，除了特定和已知的等待（例如使用 `setInterval` 或 `setTimeout` 时），**完全无法预测**休眠时间应该是多久，因为它可能取决于：

- 网络状态（对于 XHR 请求）
- 可用机器资源的总量（CPU、RAM 等）
  - 例如，CI 流水线可能会对其进行限制
  - 在本地机器上，其他应用程序也可能会消耗这些资源
- 其他资源消耗更新的并发情况（canvas 等）
- 一系列不可预测的因素，如 Service Workers、缓存管理等，可能加快或减缓 UI 更新过程

每个固定的延迟都会使测试变得更加脆弱，并**增加其持续时间**。您需要在虚假负面和夸张的测试持续时间之间找到平衡。

等待可分为四个主要类别：

- **[页面加载等待](#页面加载等待)**：测试应用程序时需要处理的第一个等待，等待一个允许您了解页面是否可交互的事件
- **[内容等待](#内容等待)**：等待匹配选择器的 DOM 元素
- **[XHR 请求等待](#xhr-请求等待)**：等待 XHR 请求开始或相应接收到

以下所有示例都基于 Cypress。

<br/><br/>

## 页面加载等待

```javascript
// Cypress code
cy.visit('http://localhost:3000')
```

## 内容等待

请看以下示例，了解如何在可用工具中实现等待 DOM 元素。

### 内容等待代码示例

- 等待元素

```javascript
// Cypress code

// it waits up to 4 seconds by default
cy.get('#form-feedback')
// the timeout can be customized
cy.get('#form-feedback', { timeout: 5000 })
```

- 等待具有特定内容的元素

```javascript
// Cypress code

cy.get('#form-feedback').contains('Success')
```

## XHR-请求等待

### XHR-请求等待代码示例

- 等待 XHR 请求/响应

```javascript
// Cypress code

cy.intercept('http://dummy.restapiexample.com/api/v1/employees').as('employees')
cy.wait('@employees')
  .its('response.body')
  .then((body) => {
    /* ... */
  })
```

<br /><br />

_由 [NoriSte](https://github.com/NoriSte) 在 [dev.to](https://dev.to/noriste/await-do-not-make-your-e2e-tests-sleep-4g1o) 和 [Medium](https://medium.com/@NoriSte/react-hooks-memorandum-bf1c2758a672)上进行联合发表._
