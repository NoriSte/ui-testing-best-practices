[✔]: assets/images/checkbox-small-blue.png

# UI Testing Best Practices (Work in progress)

<h1 align="center">
  <img src="assets/images/banner-2.jpg" alt="UI testing Best Practices">
</h1>

<br/>

<div align="center">
  <img src="https://img.shields.io/badge/⚙%20Item%20count%20-%203%20Best%20Practices-blue.svg" alt="2 items"> <img src="https://img.shields.io/badge/%F0%9F%93%85%20Last%20update%20-%20Jun%2017%202019-green.svg" alt="Last update: June 17, 2019">
</div>

<br/>

[![NoriSte](/assets/images/twitter-s.png)](https://twitter.com/NoriSte/) **Follow me on Twitter!** [**@NoriSte**](https://twitter.com/NoriSte/)

<br/>

###### Built and maintained by our [Steering Committee](#steering-committee) and [Collaborators](#collaborators)

# Latest Best Practices and News

(Work in progress)
<!-- - **New translation:** ![BR](/assets/flags/BR.png) [Brazilian Portuguese](/README.brazilian-portuguese.md) available now, courtesy of [Marcelo Melo](https://github.com/marcelosdm)! ❤️

- **New best practice:** 4.2: Include 3 parts in each test name - [_From the section "Testing and overall quality"_](https://github.com/i0natan/nodebestpractices#4-testing-and-overall-quality-practices)

- **New best practice:** 7.1: Prefer native JS methods over user-land utils like Lodash - [_From the section "Performance"_](https://github.com/i0natan/nodebestpractices#7-performance-best-practices)

- **News update:** [We kicked-off the performance section, wanna join?](https://github.com/i0natan/nodebestpractices/issues/302) -->

<br/><br/>

# Welcome! 3 Things You Ought To Know First:

**1. You are, in fact, reading dozens of the best UI testing articles -** this repository is a summary and curation of the top-ranked content on UI testing best practices, as well as content written here by collaborators

**2. It is the largest compilation, and it is growing every week -** currently, more than XX best practices, style guides, and architectural tips are presented. New issues and pull requests are welcome to keep this live book updated. We'd love to see you contributing here, whether that is fixing code mistakes, helping with translations, or suggesting brilliant new ideas. See our [writing guidelines here](/.operations/writing-guidelines.md)

**3. Most best practices have additional info -** most bullets include a **🔗Read More** link that expands on the practice with code examples, quotes from selected blogs and more information

<br/><br/>

## Table of Contents
(Work in progress, take a look at the [sections draft](/sections/draft.md) file)
1.  [Generic Best Practices (1)](#1-generic-best-practices)

# `1. Generic Best Practices`

## ![✔] 1.1 Await, don't sleep

**TL;DR:** When testing your UI, you define a sort of key points the app must pass through. Reaching these key
points is an asynchronous process because, almost 100% of the times, your UI does not update
synchronously. Those key points are called **deterministic events**, as known as something that you
know that must happen. You need to wait for these events to make your tests robust.

**Otherwise:** Sleeping the tests make your tests slow and brittle, it's one of the most common and biggest errors in UI testing.

🔗 [**Read More: Await, don't sleep**](/sections/generic-best-practices/await-dont-sleep.md)

<br/>

## ![✔] 1.2 Name your test files wisely

**TL;DR:** Lot of times you need to launch just a type of tests and it's super easy if you follow a
common pattern while naming your testing files.

**Otherwise:** You need to launch a long test suite just to have some of them run.

🔗 [**Read More: Name the test files
wisely**](/sections/generic-best-practices/name-test-files-wisely.md)

## ![✔] 1.3 Use your testing tool as your primary development tool

**TL;DR:** Leveraging your testing tool to avoid manual tests is one of the biggest improvements you
could do to speed up your working flow. Testing tools are faster than you and the most modern ones include
some UI utilities that make easy to use them as a development tool.

**Otherwise:** You code the app the old way, losing a lot of time interacting manually with the UI itself.

🔗 [**Read More: Use your testing tool as your primary development tool**](/sections/generic-best-practices/use-your-testing-tool-as-your-primary-development-tool.md)

<br/>

<br/><br/>


<br/><br/><br/>

## Steering Committee

Meet the steering committee members - the people who work together to provide guidance and future direction to the project.

<img align="left" width="100" height="100" src="assets/images/members/noriste.png">

[Stefano Magni](https://github.com/NoriSte)
<a href="https://twitter.com/NoriSte"><img src="assets/images/twitter-s.png" width="16" height="16"></img></a>
<a href="https://www.linkedin.com/in/noriste/"><img src="assets/images/linkedin.png" width="16" height="16"></img></a>
<a href="https://stackoverflow.com/users/story/700707"><img src="assets/images/stackoverflow.png" width="16" height="16"></img></a>

A positive-minded front-end developer. He's a passion for good UIs, automation, testing and teaching. He's developed every kind of interface: webapps, mobile apps, smartTV apps and games.

<br/>



## Collaborators

Thank you to all our collaborators! 🙏

Our collaborators are members who are contributing to the repository on a reguar basis, through suggesting new best practices, triaging issues, reviewing pull requests and more. If you are interested in helping us guide thousands of people to craft better UI tests, please read our [contributor guidelines](/.operations/CONTRIBUTING.md) 🎉

<br/>

## Thank You Notes

We appreciate any contribution, from a single word fix to a new best practice. Below is a list of everyone who contributed to this project. A 🌻 marks a successful pull request and a ⭐ marks an approved new best practice.

### Flowers

A successfull PR gives you a 🌻, be the first to collect it.

### Stars

An approved new best practice Be the first to collect a ⭐, contribute to this repository 😁

<br/><br/><br/>

This repository is inspired to the [nodebestpractices](https://github.com/i0natan/nodebestpractices) one, thank you [Yoni](https://github.com/i0natan) and the whole [steering team](https://github.com/i0natan/nodebestpractices#steering-committee) to keep it updated and to allow the creation of this repository.

<br/><br/><br/>
