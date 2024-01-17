# 测试监控

<br/><br/>

## 一段简要说明

随着前端期望的提高，服务器和服务的复杂性也在增加。前端应用需要变得越来越快：代码拆分、懒加载、Brotli 压缩等性能优化解决方案已成为标准。还有一些令人惊叹的解决方案，如基于机器学习和分析数据的代码拆分和资源预加载。此外，JAMstack 站点生成器可用于避免手动管理许多性能优化，但它们的配置和构建过程可能会破坏**已经测试过的功能**。

有很多我们一旦测试过就认为理所当然的功能，但它们并非无法回归，可能会导致灾难。例如：

- `sitemap.xml` 和 `robots.txt` 的爬行配置（通常每个环境都不同）
- 使用 Brotli/gzip 提供的资产：错误的内容编码可能会破坏站点的所有功能
- 针对静态或动态资产的不同配置的缓存管理

这些问题可能看起来很明显，但比你想象的更微妙。如果你关心用户体验，就应该保持监控，因为通常在发现问题时已经为时过晚。我知道不能监控所有事物，但是测试应用的次数越多，测试就越能成为首要的质量检查工具。

监控测试也可以与 E2E 测试集成（毕竟，它们只是简单的 E2E 测试），但将它们保持分开可以帮助你在需要时运行它们。上述大多数事项与 DevOps 相关，有了超快的监控测试，您可以获得即时而专注的反馈。

<br/><br/>

## Cypress 示例

- 缓存监控测试示例

```javascript
const urls = {
  staging: "https://staging.example.com",
  production: "https://example.com",
}

const shouldNotBeCached = (xhr) => cy.wrap(xhr).its("headers.cache-control").should("equal", "public,max-age=0,must-revalidate")
const shouldBeCached = (xhr) => cy.wrap(xhr).its("headers.cache-control").should("equal", "public,max-age=31536000,immutable")
// extract the main JS file from the source code of the page
const getMainJsUrl = pageSource => "/app-56a3f6cb9e6156c82be6.js"

context('Site monitoring', () => {
  context('The HTML should not be cached', () => {
    const test = url =>
      cy.request(url)
        .then(shouldNotBeCached)

    it("staging", () => test(urls.staging))
    it("production", () => test(urls.production))
  })

  context('The static assets should be cached', () => {
    const test = url =>
      cy.request(url)
        .its("body")
        .then(getMainJsUrl)
        .then(appUrl => url+appUrl)
        .then(cy.request)
        .then(shouldBeCached)

    it('staging', () => test(urls.staging))
    it('production', () => test(urls.production))
  })
})
```

- 内容编码监控测试示例

```javascript
context('The Brotli-compressed assets should be served with the correct content encoding', () => {
  const test = url => {
    cy.request(url)
    .its("body")
    .then(getMainJsUrl)
    .then(appUrl => cy.request({url: url + appUrl, headers: {"Accept-Encoding": "br"}})
      .its("headers.content-encoding")
      .should("equal", "br"))
  }

  it('staging', () => test(urls.staging))
  it('production', () => test(urls.production))
})
```

- 抓取监测测试示例

```javascript
context('The robots.txt file should disallow the crawling of the staging site and allow the production one', () => {
  const test = (url, content) =>
    cy.request(`${url}/robots.txt`)
      .its("body")
      .should("contain", content)

  it('staging', () => test(urls.staging, "Disallow: /"))
  it('production', () => test(urls.production, "Allow: /"))
})
```

<br /><br />

*由[NoriSte](https://github.com/NoriSte) 在 [dev.to](https://dev.to/noriste/the-concept-of-monitoring-tests-4l5j) 和 [Medium](https://medium.com/@NoriSte/the-concept-of-monitoring-tests-d7cb5af514e5)进行联合发表.*
