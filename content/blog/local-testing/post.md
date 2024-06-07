---
title:  "Developer Experience starts locally"
date:   2023-11-09
authors:
  - name: Kaj Fehlhaber
tags:
  - developer experience
  - platform engineering
  - testing
---
If you are going to develop an application of sort and you are provided with an environment where you can deploy it -
what would be the first thing that you want working well in your developer experience?

You should answer: "to be able to quickly test things out locally - and it should just work". Otherwise this blog post
would need a new name... 😄

The point is: Feedback is key! You have probably heard this a thousand times by now, but it still needs re-iterating
since I see the lack of built-in support for quick feedback over and over.
The question is how this can be implemented. Luckily, with a cloud native setup, even a full service can be tested with
relative ease.

Let's dive into some areas. I will, however, skip regular unit testing and such since this should already be very mature
how this is achieved.

## It works on my cluster
Kubernetes is hard. Or so they say. Well, OK - there is quite a lot to learn around it. But the good thing is that there
is not so much needed for an application developer to know to be effective and to troubleshoot a service. Platform
engineers can manage clusters and configure it in a way which makes it secure, that it can scale and also provide
necessary tooling for observability and a lot more.

What might **not** be discussed as often is that any Kubernetes workload can be very effectively tested locally in for
example a [`kind`](https://kind.sigs.k8s.io/) (single node) cluster. You can literally re-create your whole cluster
together with an entire observability stack, messaging service and persistence, together with you apps - in matter of
just a few minutes or less.

And since you are practicing GitOps, the local cluster is setup just like any other stage with the exception that you
likely won't run any CD tool.

### Use open interfaces and tools!
I'm the first to say that I once thought that persistence is something you should really use a managed service to
handle...

...but things change. Platform engineering practices and the cloud native landscape has literally exploded with a very
active community being behind big steps when it comes to for example operators for stateful services.

I just recently wrote a blog post about [Managed services](../managed-services/post) where some side effects of managed
services where discussed. When using managed services, open api:s should also be used - for the reason of local testing.
If using proprietary api:s, there will always be a tax to the local testing experience.
But you could get away from this altogether by using open api:s and even better - to deploy your services 
