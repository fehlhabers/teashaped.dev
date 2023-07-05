---
layout: post
title:  "Story of a Story about Minimal Viable Change"
date:   2023-07-05
cover_image: story-of-story/mvp.png
author: Kaj Fehlhaber
categories: mvp,testing,story-slicing,yagni
---
Let me take you through the work process of a story we just worked with...

I do this because I want to both challenge the view of testing, story slicing and the ever reoccuring plague of over- and pre-optimization, as well as giving some concrete examples of ways to break work apart in smaller pieces. I won't go deep into details, but try to raise relevant parts in every step. Let's check it out!

## Unofficial ground rules
A lot of the upcoming steps will have some rules or topics in common, namely:

- Set small completable goals which leaves you with everything merged and in a deployable state
- ABSOLUTELY no pre-optimization!
- Don't be dogmatic about ANY discipline (like TDD, "clean code", etc)
- Do just the right amount of tests and type of tests
- Automate identified bottlenecks

How would this look in practice? Let's go through it.

**But first some short overview of what we were about to develop:**

The feature required us to interface with an identity provider and an external endpoint.
It would be the first feature in our service which requires accessing external endpoints and as a result we would also like to be able to mock the service for our automated system tests. A capability we do not yet have.
The feature would be an update to an existing functionality, so it could break the existing service.
Our service would run in parallel with another service, so the external systems should not be called more than for specific conditions.

How would you go about it? How do you introduce the external dependency without a big bang? And how do you setup the testing?

## Our steps

### Create client library
Since we have two new interfaces where one *will* be reused and is a prereq of the other, we start out by creating a thin library client for it.
Since it's an identity provider using oauth, we want to implement it as a read-through cache. We unit test it and achieve our first goal. We feel confident enough with unit tests at this stage, since it is a stable interface and we have worked with it before.

### Create new component
We create a component which handles the new flow. We don't know if the interface is going to be used anywhere else, so we build it right into the application. We unit test the new component, which utilizes the library built previously. Another goal achieved.

### Use new component with feature toggle
Since we already have component tests setup, we wait with a real integration test until we have connected the new feature. We now make use of the new component in our flow, but make sure to use *feature toggling* for the feature. We update automated component tests with mocks for the new endpoint. All config about external systems are injected to the component, so we can easily point to the right system. Another goal achieved.

### Manual integration tests
Manual testing against real endpoints done using altered component tests which involved the entire app. Minor adjustments made for the typical integration issues found. Feature toggle turned off and merged. CI/CD builds and deploys.

### Mock server
Our goals is that most our automated system tests use mocked external systems to ensure we have control over our tests and avoid flakiness.
Therefore, next step is to be able to enable the feature using only mock server first.
We create a very simple web server with endpoints mocking the real servers, add to our pipeline and merge. CI/CD build and deploys.

### Feature only uses mock server
We update our helm charts to pass in the mock server URL:s to the app and enable the feature flag in dev environment. Merge and CI/CD deploys. System tests remain green.

### Mock server capable of capturing requests
We want to assert that the feature calls the relevant external sources, so we use middleware in the mock server to collect all incoming requests and expose a new endpoint for our system test to get captured requests by the tracking-id related to the test run. We simply store incoming requests in-memory and don't add any cleanup logic. We do that if the need arises. We add simple component tests to verify behavior and merge. CI/CD builds and deploys.

### System tests assert using mock server
We add another step to our existing test scenario, where we use the new mock server endpoint to assert that a proper request was made. Since we are able to run our system tests from our local environment, we can quickly test that the new additions work and merge = deploy. The full flow now works, but only uses mock server. The process to setup local testing felt clumsy, so it was automated and a README was added about the setup (which was now one command).

### Enable real endpoints
Since it's important not to call the real endpoints other than for specific conditions to start with (same as canary), we wrap the new component with the toggling functionality. We can inject both mock and real URL:s as well as secrets. The component can remain unchanged. We see that we need a system wide solution for dynamic feature toggling, but do not need it for this feature. We implement a simple solution now and create a new item in Jira about the need of the new functionality.
System tests are updated with one E2E test using the real endpoint (which meats the conditions to use the real endpoints) and another which is identified as _test_ and thus routed to mock server. DONE!

## Wrapping up
Throughout the story we could merge good sized PR:s to main and all of the time keep main in a deployable state. This was achieved by setting small goals throughout the work and utilizing techniques such as feature toggling and by building testable components.

All of this was actually one story and one could argue that this was quite a big story and that we should make them smaller. I could agree with this, but it also fits well with the idea of not changing things that work well and instead focus on parts that don't. We work in ensembles and have been able to estimate very well. Planning and breaking down stories too early is also something better avoided. It more often leads to the same discussions being repeated multiple times, why we choose not to refine too deep.

I hope this story was interesting for you and gave you some fruit for thought!

<br>
<br>
<br>
![](/assets/images/kaj.jpg){:style="margin-left: 0px;margin-right: auto;"}
**Kaj Fehlhaber**<br>
Freelancing Software Consultant<br>
_Accelerating the DevOps Journey of Teams!_
