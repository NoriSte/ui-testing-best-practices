# Monitoring tests

<br/><br/>

### One Paragraph Explainer

The more the front-end expectations grow, the more the server and services complexity grows. Front-end applications need to be faster every day: code splitting, lazy loading, brotli compression and a lot of other performance-oriented solutions became a standard during the last years. Not to cite some amazing solutions like code splitting and resources preloading based on machine learning and analytics data. More: JAMstack site generators are useful to avoid manually managing a lot of performance optimizations but their configuration and their build processes could break an **already tested feature**.

There are a lot of features that we take for granted once tested that are not regression-free and that could lead to disasters. Some examples could be:
- `sitemap.xml` and `robots.txt` crawling configurations (usually different for every environment)
- Brotli/gzip served assets: a wrong content-encoding could break all the site functionalities
- cache management with different configurations for static or dynamic assets

They could seem obvious things but they are more subtle then what you think. If you care about the good user experience you should keep it monitored because it's usually too late when you discover that something did not work. You could not monitor everything, I know, but the more you test your applications, the more you leverage tests as your first quality checker.

Monitoring tests could also be integrated with E2E ones (after all, they are super-simple E2E tests) but keeping them separated helps you running them-only if you need. Most of the above-cited things are DevOps-related and having super-fast monitoring tests allows you to have immediate and focused feedbacks.

<br/><br/>

## Cypress examples

Cache monitoring tests

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

Content encoding monitoring tests

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

Crawling monitoring tests
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

*Crossposted by [NoriSte](https://github.com/NoriSte) on [dev.to](https://dev.to/noriste/the-concept-of-monitoring-tests-4l5j) and [Medium](https://medium.com/@NoriSte/the-concept-of-monitoring-tests-d7cb5af514e5).*
