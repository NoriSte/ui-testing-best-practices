# CI CD strategies for UI apps and deployed services

In today's rapidly-evolving digital ecosystem, Continuous Integration and Continuous Deployment (CI/CD) strategies have become integral to the lifecycle of web applications and services. CI/CD offers a plethora of benefits that enable organizations to deliver products to market rapidly, streamline their operations, and drastically improve their overall software quality. CI/CD is the heartbeat of modern DevOps practices. It bridges the gap between development and operations teams, fostering a culture of seamless collaboration. This synergy is underpinned by a fail-fast mentality, where errors are detected and corrected promptly during the development process, rather than after deployment.

Without CI/CD, software development becomes a cumbersome, time-consuming endeavor fraught with bottlenecks and pitfalls. This traditional, siloed approach to software development and deployment—known as the waterfall model—often leads to "integration hell", where merging changes from different team members becomes a nightmare, causing delays, and potentially leading to flawed final products. In a world that demands agility and reliability, the absence of CI/CD strategies can make your UI apps and deployed services lag in the competitive landscape.

In this article, we'll explore two distinct examples of CI/CD implementations, each demonstrating a unique scenario. Our first example centers on a React application, while the second focuses on a service deployed on Amazon Web Services (AWS). For both cases, we will be utilizing GitHub Actions as our CI/CD platform and sharing the repos, giving us the advantage of exploring actual repositories to deepen our understanding.

Remember, there is no one-size-fits-all solution when it comes to CI/CD. However, the overarching principles and strategies we'll discuss in these examples hold universal applicability. They can be adapted and modified to fit a variety of different situations and needs

- [CI CD strategies for UI apps and deployed services](#ci-cd-strategies-for-ui-apps-and-deployed-services)
  - [CI CD for a UI app](#ci-cd-for-a-ui-app)
    - [Key Concepts](#key-concepts)
    - [Testing Against Localhost on PRs, Testing Against Deployments](#testing-against-localhost-on-prs-testing-against-deployments)
  - [CI CD for a deployed AWS service](#ci-cd-for-a-deployed-aws-service)
    - [Testing against temporary branches on PRs](#testing-against-temporary-branches-on-prs)
      - [Key Concepts:](#key-concepts-1)
    - [Testing against deployments](#testing-against-deployments)
  - [Wrap up](#wrap-up)

## CI CD for a UI app

We'll start by examining a [Tour of Heroes repo](https://github.com/muratkeremozcan/tour-of-heroes-react-vite-cypress-ts), featured in the book [CCTDD: Cypress Component Test Driven Design](https://muratkerem.gitbook.io/cctdd/). This repository demonstrates various test checks, including lint (ESLint), type checks (TS), unit tests (Jest), and Cypress component tests & end-to-end tests. The tests are parallelized to reduce feedback time to approximately five minutes, which is an optimal duration to promote a continuous feedback loop for this size of repo.

Below is the CI architecture for this project:

![CICD-ui-app](/assets/images/cicd/CICD-ui-app.png)

The app contains 22 Cypress component tests, 22 Jest/RTL unit/component tests, and 11 Cypress end-to-end tests. With [Cypress Cloud analytics](https://cloud.cypress.io/projects/x953wq/runs/831/specs), there is potential to further reduce this time.

![Cy-Cloud](/assets/images/cicd/Cy-Cloud.png)

### Key Concepts

- Dependencies are installed and cached, drastically reducing the setup time for subsequent commits.
- Unit tests, linting, and type checks are all parallelized, taking advantage of the cache.
- Component tests and end-to-end tests are parallelized as well, without waiting for unit tests, linting, and type checks to complete.
- Nine machines run in parallel, speeding up the CI pipeline. If we have limited CI machines, we could run the Cypress parallelization sequentially after the linting, type check, and unit test stage, but it would increase feedback time by about 25%.
- End-to-end tests execute against `localhost:<port-number>`, which is served locally on the CI machine. This approach, often overlooked, enables a shift-left testing strategy that we'll discuss more later.

This [YAML file](https://github.com/muratkeremozcan/tour-of-heroes-react-vite-cypress-ts/blob/main/.github/workflows/main.yml) details the CI implementation, including combined code coverage with [CodeCov](https://about.codecov.io/). For a simpler example without Cypress parallelization and code coverage, check the [Github Actions YAML file of this template](https://github.com/muratkeremozcan/react-cypress-ts-vite-template/blob/main/.github/workflows/main.yml). The ideas presented here can be applied to any front-end application.

```yml
name: unit-lint-typecheck-e2e-ct
on:
  push:
  workflow_dispatch:

# to cancel outdated runs when a new commit in the same branch comes in
concurrency:
  group: ${{ github.head_ref || github.ref }}
  cancel-in-progress: true

jobs:
  install-dependencies:
    name: Install Dependencies
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v3
        with:
          ref: ${{ github.event.pull_request.head.sha }}

      # we use the Cypress GHA to cache dependencies,
      # and we do not run the tests
      - name: Install dependencies
        uses: cypress-io/github-action@v5.8.0
        with:
          runTests: false

  # we have all the jobs below need the dependencies
  # so that they run sequentially

  # unit, lint and typecheck jobs all install dependencies the same way;
  # they all hit the cache from the dependencies job

  unit-test:
    needs: [install-dependencies]
    name: Run Unit Test
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v3
        with:
          ref: ${{ github.event.pull_request.head.sha }}

      - name: Install dependencies
        uses: cypress-io/github-action@v5.8.0
        with:
          runTests: false

      - name: unit-test
        run: yarn test:

  lint:
    needs: install-dependencies
    name: Run Lint
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v3
        with:
          ref: ${{ github.event.pull_request.head.sha }}

      - name: Install dependencies
        uses: cypress-io/github-action@v5.8.0
        with:
          runTests: false

      - name: lint
        run: yarn lint

  typecheck:
    needs: install-dependencies
    name: Run typecheck
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v3
        with:
          ref: ${{ github.event.pull_request.head.sha }}

      - name: Install dependencies
        uses: cypress-io/github-action@v5.8.0
        with:
          runTests: false

      - name: typecheck
        run: yarn typecheck

  cypress-e2e-test:
    needs: [install-dependencies]
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v3
        with:
          ref: ${{ github.event.pull_request.head.sha }}

      - name: Cypress e2e tests 🧪
        uses: cypress-io/github-action@v5.8.0
        with:
          start: yarn start
          wait-on: 'http://localhost:3000'
          browser: chrome
        env:
          # enables unique runs (when the dreaded retry button is used...)
          GITHUB_TOKEN: ${{ secrets.GITHUB_TOKEN }}

  cypress-ct-test:

    needs: [install-dependencies]
    runs-on: ubuntu-latest

    steps:
      - uses: actions/checkout@v3
        with:
          ref: ${{ github.event.pull_request.head.sha }}

      - name: Cypress component tests 🧪
        uses: cypress-io/github-action@v5.8.0
        with:
          component: true
          browser: chrome
        env:
          GITHUB_TOKEN: ${{ secrets.GITHUB_TOKEN }}

```

### Testing Against Localhost on PRs, Testing Against Deployments

As alluded to in the previous section, testing against localhost during the Pull Request (PR) stage is a cornerstone of the shift-left strategy. This approach ensures that no developer merges changes into the mainline branch unless all CI checks pass.

In our 'Tour of Heroes' example, this requirement extends to combined code coverage. If a new PR decreases the coverage rate below 100% (for instance, by introducing new code without associated tests), the CI pipeline will fail.

This rigorous testing practice not only encourages code quality but also fosters a paradigm shift towards process automation. Consequently, it reduces the reliance on management tools for process enforcement, thus improving efficiency and developer experience.

But what happens after the PR is merged? What about continuous deployment? For this, let us examine another UI app which employs Amplify for deployments: Yan Cui's [Vue-based Twitter clone](https://github.com/muratkeremozcan/appsyncmasterclass-frontend/tree/main/.github/workflows). Looking at this repository, we find two YAML files. [`PR.yml`](https://github.com/muratkeremozcan/appsyncmasterclass-frontend/blob/main/.github/workflows/PR.yml) is very similar to our previous Tour of Heroes example, though this application is built with Vue & JavaScript rather than React & TypeScript. Noticeably, it only runs on PRs.

```yml
# ./.github/workflows/PR.yml
name: PR
on:
  pull_request: # only PRs
  workflow_dispatch: # we can manually trigger...

# the rest of the file...
```

The more interesting part is the [ `dev.yml`](https://github.com/muratkeremozcan/appsyncmasterclass-frontend/blob/main/.github/workflows/dev.yml) file, which only runs the e2e tests, but this time with a custom config file for dev deployment. Line 31 of the yml specifies the config file:

```yml
# ./.github/workflows/dev.yml
name: dev
on:
  push:
    branches: [main] # only on main
  workflow_dispatch: # manual trigger...

jobs:
  cypress-e2e-test:
    # generic settings...
    steps:
      # generic settings...

      - name: Cypress e2e tests 🧪
        uses: cypress-io/github-action@v5.0.8
        with:
          # KEY point; we specify what config file to use
          config-file: cypress/config/dev.config.js

        # generic settings...
```

When running tests against multiple deployments, it is a best practice to have separate config files for each deployment, usually differentiating by the baseUrl.

```js
// ./cypress/config/local.config.js

const { defineConfig } = require("cypress");

module.exports = defineConfig({
  // generic settings...

  e2e: {
    setupNodeEvents(on, config) {
      // generic settings...
    },
    // we have some custom env vars
    env: {
      ENVIRONMENT: "dev",
      API_URL:
        "https://awfrp7n7rrhw5kzqimfegnqzeq.appsync-api.eu-west-1.amazonaws.com/graphql",
    },
    // we have a baseUrl for dev deployment
    baseUrl: "https://main.d2uw1pp8i1hsae.amplifyapp.com/#/",
  },

  // component tests only apply to PRs/local,
  // some additional settings for it here
  component: {
    devServer: {
      framework: "vue-cli",
      bundler: "webpack",
    },
  },
});
```

```js
// ./cypress/config/dev.config.js

const { defineConfig } = require("cypress");

module.exports = defineConfig({
  // generic settings...

  e2e: {
    setupNodeEvents(on, config) {
      esbuildPreprocessor(on);
      registerDataSession(on, config);
      return config;
    },
    // UI PRs use the same dev api env vars
    env: {
      ENVIRONMENT: "dev",
      API_URL:
        "https://awfrp7n7rrhw5kzqimfegnqzeq.appsync-api.eu-west-1.amazonaws.com/graphql",
    },
    // our base url for PRs
    baseUrl: "http://localhost:8080/#/",
  },
});
```

As for the Amplify specifics, there is an `amplify.yml` file in the repository and AWS Amplify is configured to recognize our repository. Several options are available for Continuous Deployment of UI applications. Apart from AWS Amplify, Netlify, Vercel, Google Firebase, Heroku, GitHub Pages, Azure App Service are just a few of the other platforms. Each typically requires a configuration file in the repository and a web app where a project is created and linked to a repository.

However, as these configurations are vendor-specific, we will instead delve into continuous deployment details using an AWS service example with temporary branches or ephemeral instances. (Perhaps in the future we will take the ToH and deploy it via multiple vendors for a through comparison.)

## CI CD for a deployed AWS service

In this section, we will explore the process of implementing Continuous Integration and Continuous Deployment (CI/CD) for a service deployed on AWS. We will be using Yan Cui's [AWS AppSync Twitter clone](https://github.com/muratkeremozcan/appsyncmasterclass-backend) as a sample.

> For the record, AWS AppSync is a fully managed service that enables developers to develop GraphQL APIs with ease, and this service uses GraphQL instead of a more wide spread API gateway.

Of interest are 3 yml files under [./github/workflows/](https://github.com/muratkeremozcan/appsyncmasterclass-backend/tree/main/.github/workflows); `PR.yml`, `dev.yml`, `stage.yml`. Let us work through [ `PR.yml`](https://github.com/muratkeremozcan/appsyncmasterclass-backend/blob/main/.github/workflows/PR.yml)

> Disclaimer: Yan uses Jest for unit (mocks AWS resources), integration (unit test-like, uses real AWS resources) and e2e tests. We took this opportunity to create Cypress mirrors of the e2e tests, and discussed the style further [in this video](https://www.youtube.com/watch?v=jTgT3VGhqKw). Therefore, we are less worried about parallelization, or running Jest tests separately for unit, integration or e2e. We just want a serial job set, and the main focus is the deployment and the removal of the temporary stack.

### Testing against temporary branches on PRs

A critical aspect of CI/CD at PR level is the ability to create and test against temporary branches with Serverless Framework. Each feature branch is deployed to a dedicated environment, like `my-feature`, and the temporary branch's name is typically the same as the git branch (with the `/` character avoided or replaced).

> If we have external services that are not a part of our stack, or serverful resources we would like to keep outside of our stack, we do not include these serverful resources as part of the ephemeral environments and would share to those resources, and/or refer to their dev deployments on our PRs. Check out Yan Cui's blog post [How to handle serverful resources when using ephemeral environments](https://theburningmonk.com/2023/02/how-to-handle-serverful-resources-when-using-ephemeral-environments/) for a through explanation.

Deploying a temporary branch using the [Serverless framework](https://serverless.com/framework/) is simple with the command `sls deploy -s my-feature`. Once the feature has been fully tested and integrated, the temporary stack can be removed using `sls remove -s dev-my-feature`.

> SAM (Serverless application model), CDK (Cloud Development Kit), Amplify CLI, Terraform are other infra as code alternatives to Serverless Framework, and we assume each has the capability to deploy and remove stacks in some shape or form. Temporary branches are just so fundamental and easy in Serverless Framework, therefore it is being used in the examples.

#### Key Concepts:

- To use the Serverless Framework, we need to install the AWS CLI.
- A unique name for our temporary stack is generated from the branch name.
- Generic steps like linting and testing are performed.
- The temporary branch is deployed before running e2e tests against it.
- The temporary stack is removed at the end of the run.

Temporary branches are regarded as crucial in serverless engineering. They allow engineers to develop and test in isolated environments and stacks without clashing with each other on a shared deployment. This setup also helps avoid having to clean up test data, but mind data if tests run on deployments test data still has to be cleared.

```yml
name: deploy to temp stack

on:
  pull_request: # runs on branches
  workflow_dispatch:

concurrency:
  group: ${{ github.head_ref || github.ref }}
  cancel-in-progress: true

jobs:
  build:
    runs-on: ubuntu-latest

    # env vars so Serverless Framework can deploy and remove the stack
    env:
      AWS_ACCESS_KEY_ID: ${{ secrets.AWS_ACCESS_KEY_ID }}
      AWS_SECRET_ACCESS_KEY: ${{ secrets.AWS_SECRET_ACCESS_KEY }}
      AWS_DEFAULT_REGION: eu-west-1

    steps:
      # Checks-out your repository under $GITHUB_WORKSPACE, so your job can access it
      - uses: actions/checkout@v3
      # needed to install AWS CLI, which we need for Serverless Framework
      - uses: actions/setup-python@v4
        with:
          python-version: "3.x"
      - uses: actions/setup-node@v3
        with:
          node-version: "16"

      # generic step
      - name: Install dependencies
        uses: cypress-io/github-action@v5.6.1
        with:
          install-command: npm ci --force
          runTests: false

      - name: Install AWS CLI
        run: |
          curl "https://awscli.amazonaws.com/awscli-exe-linux-x86_64.zip" -o "awscliv2.zip"
          unzip awscliv2.zip
          sudo ./aws/install --update

          rm awscliv2.zip
          rm -r ./aws

      # generic step
      - name: Lint
        run: npm run lint

      # since we want to deploy a temporary stack with the branch name,
      # we have to get the branch name
      - name: Get branch name
        id: branch-name
        uses: tj-actions/branch-names@v6

      # command to deploy the stack,
      # the same idea as deploying the stack from our laptop
      - name: deploy to ${{ steps.branch-name.outputs.current_branch }}
        run: |
          npm run sls -- config credentials --provider aws --key ${{ secrets.AWS_ACCESS_KEY_ID }} --secret ${{ secrets.AWS_SECRET_ACCESS_KEY }} --overwrite     
          npm run sls -- deploy -s ${{ steps.branch-name.outputs.current_branch }}

      # some convenience so that we can use process.env in our code and tests
      - name: export env vars
        run: |
          npm run sls export-env -- -s ${{ steps.branch-name.outputs.current_branch }}

      # all jest tests; unit, integration, e2e
      - name: jest tests
        run: npm t

      # Cypress e2e
      - name: Cypress e2e tests 🧪
        uses: cypress-io/github-action@v5.6.1
        with:
          browser: chrome
          install: false
        env:
          GITHUB_TOKEN: ${{ secrets.GITHUB_TOKEN }}

      # clean up the stack
      - name: Remove stack ${{ steps.branch-name.outputs.current_branch }}
        run: |
          npm run sls -- remove -s ${{ steps.branch-name.outputs.current_branch }}
```

Here is [a sample run](https://github.com/muratkeremozcan/appsyncmasterclass-backend/actions/runs/4667128107), taking under 10 minutes. The biggest time consumption is the deployment taking under 5 minutes which is the worst case scenario since this is a new stack. We can reduce this by half if we use [lambda layers](https://github.com/muratkeremozcan/appsyncmasterclass-backend#79-serverless-layers-to-reduce-package-size). Another one is Removing the stack taking 1:41. We could instead only remove the stack once the PR is merged (but currently we do not know how to handle this with GHA - let us know in the comments if you do). Finally, the tests take a serial 1:33 with Jest and 51s with Cypress, which we could save a minute with parallelization. This is a beefy service with many tests and we could reduce the feedback time to around 5 minutes. Albeit, around 10 minutes CI feedback is fair when deploying stacks from scratch and removing them because temporary stacks is considered the holy grail of e2e testing deployed services.

> Note about config file and environment variables: in this service, the environment variables are unique per deployment, meaning our dozen+ environment variables change values with each branch, including the API url. For this reason, we have a step to export environment variables in CI, and this is also a requirement when working locally. This means we cannot have unique config files for Cypress, but we can map `process.env` to Cypress `env` [like so](https://github.com/muratkeremozcan/appsyncmasterclass-backend/blob/main/cypress.config.js#L11). Check out the video [map your .env file to Cypress environment variables](https://www.youtube.com/watch?v=fq-VDY6VQls) for a demo.

### Testing against deployments

Next, let us look at [`dev.yml`](https://github.com/muratkeremozcan/appsyncmasterclass-backend/blob/main/.github/workflows/dev.yml). This file simplifies the process for branches as it doesn't need to lint, unit test, or remove the branch. The deployments are persistent and environment variables do not change frequently, making the overall testing simpler. We can see that [a sample run](https://github.com/muratkeremozcan/appsyncmasterclass-backend/actions/runs/5120215093) takes under 5 minutes because we have deployed to dev before and we do not remove it at the end.

```yml
# /.github/workflows/dev.yml

name: deploy to dev

on:
  push:
    branches: [main] # runs on main branch, upon merging a PR
  workflow_dispatch:

concurrency:
  group: ${{ github.head_ref || github.ref }}
  cancel-in-progress: true

jobs:
  build:
       runs-on: ubuntu-latest

    env:
      AWS_ACCESS_KEY_ID: ${{ secrets.AWS_ACCESS_KEY_ID }}
      AWS_SECRET_ACCESS_KEY: ${{ secrets.AWS_SECRET_ACCESS_KEY }}
      AWS_DEFAULT_REGION: eu-west-1

    steps:
      - uses: actions/checkout@v3
      - uses: actions/setup-python@v4
        with:
          python-version: '3.x'
      - uses: actions/setup-node@v3
        with:
          node-version: '16'

      - name: Install dependencies
        uses: cypress-io/github-action@v5.6.1
        with:
          install-command: npm ci --force
          runTests: false

      - name: install AWS CLI
        run: |
          curl "https://awscli.amazonaws.com/awscli-exe-linux-x86_64.zip" -o "awscliv2.zip"
          unzip awscliv2.zip
          sudo ./aws/install --update

          rm awscliv2.zip
          rm -r ./aws

      - name:
          deploy to dev
        run: |
          npm run sls -- config credentials --provider aws --key ${{ secrets.AWS_ACCESS_KEY_ID }} --secret ${{ secrets.AWS_SECRET_ACCESS_KEY }} --overwrite
          npm run sls -- deploy

      - name: export env vars
        run: npm run export:env

      # no need for integration and unit tests on dev, they get covered on PRs
      - name: Cypress e2e tests 🧪
        uses: cypress-io/github-action@v5.6.1
        with:
          browser: chrome
          install: false
        env:

          GITHUB_TOKEN: ${{ secrets.GITHUB_TOKEN }}
```

There are only 3 differences in the [`stage.yml`](https://github.com/muratkeremozcan/appsyncmasterclass-backend/blob/main/.github/workflows/stage.yml) file. We run it when pushing tags. We specify to deploy to stage with `npm run sls -- deploy -s stage `. And finally, the script for exporting stage environment variables is slighly different.

```yml
name: deploy to stage

on:
  push:
    tags: ["*"] # runs when we push tags
  workflow_dispatch:

jobs:
  build:
    runs-on: ubuntu-latest
    env:
      # same as dev.yml

    steps:
      # all steps until her are the same
      # the key difference is specifying the stage with "-s stage"
      - name: deploy to stage
        run: |
          npm run sls -- config credentials --provider aws --key ${{ secrets.AWS_ACCESS_KEY_ID }} --secret ${{ secrets.AWS_SECRET_ACCESS_KEY }} --overwrite     
          npm run sls -- deploy -s stage

      # and the script for exporting stage environment variables is slighly different
      - name: export env vars
        run: npm run export:env-stage

      # e2e step is still the same
      - name: Cypress e2e tests 🧪
        uses: cypress-io/github-action@v5.6.1
        with:
          browser: chrome
          install: false
        env:
          GITHUB_TOKEN: ${{ secrets.GITHUB_TOKEN }}
```

## Wrap up

In conclusion, a well-structured CI/CD pipeline allows us to integrate and deploy code changes rapidly and reliably, facilitating an agile development process. Utilizing the right tools and practices is crucial to ensure the process is efficient and effective.

Through the examples we discussed, we explored how these principles apply in different contexts. We saw the importance of engineers working in unique, isolated environments through the use of temporary branches in the AWS service example, and using `localhost` in UI apps. We highlighted the importance of comprehensive testing - linting, type checking, unit tests, and end-to-end (e2e) tests, in the case of two UI apps.

These practices provide timely feedback and enable deployments to production in a minimal number of steps. Whether dealing with PRs, deployments, or AWS services utilizing temporary branches, the key concepts remain similar and crucial for a successful CI/CD pipeline.

By adhering to these best practices, we can ensure a robust and efficient pipeline that accelerates development while maintaining the high quality of the software.
