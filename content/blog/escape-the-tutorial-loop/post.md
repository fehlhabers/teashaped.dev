---
title:  "Escaping the tutorial/analysis loop"
date:   2024-03-20
authors:
  - name: Kaj Fehlhaber
tags:
  - habits
  - exploration
draft: true
---
![Cover Image](cover.jpg)
Have you ever felt that you've been watching one YouTube tutorial after another and get inspired, but the never really
getting to a point where you feel you get better? Or maybe it's about learning something new and you've been browsing
through half a Udemy course to understand the concept of it?

What if I told you there is a better way? Sure, there are good videos that explain concepts out there, but you need to
add another element to the equation for it to have any effect:

**Practice**

Or, in other words - instead of discussing a concept, you can just practice it.

## The "design phase"
This applies to both individuals and teams alike. 
For teams, this can show itself by the team, before building anything, goes to lengthy discussions about
what type of technology the new service should be built in. If they are building the apps in a hexagonal architecture,
what database is the best and that they of course need the latest and greatest framework (just because they "need" it).

For individuals, it can really be the same. You just don't have anyone else to discuss it with but yourself. That can be
enough already in order to get stuck in paralysis.

## Escaping the habit
You might have heard the mantra:

**Ship often and get feedback from your customers**

The same applies internally. Team and individuals alike.
What you build doesn't have to be perfect. The whole point is get feedback from yourself on what you've built. That is -
the journey to get there is what matters. You created it and you gained experience along the way.

## Hypothesising & Prototyping
The next time you end up in a discussion in your team and you feel you don't get forward on what design or technology
you are choosing: Think about how quick it would be to prototype it. Maybe it would be faster to actually create
something to test? Or maybe even create two prototypes and test them.

In our team we seldom go more than 30 minutes into a discussion before someone mentions that it's time to stop the
discussion and create something. At this phase there is almost always at least a hypothesis or concept which can be
tried out.

## Concrete examples

### Learning Kubernetes
Let's say that you want to learn to work with Kubernetes, but you don't have any experience with it. How would you
approach it?

#### 1. Do some overview research
Before trying to learn something you of course need to know **what it is**. Otherwise there would not be a particular
driver behind you learning it. Choose areas which make sense for you and which can help you out with what you do right
now.

At this point you hopefully know that Kubernetes is used to orchestrate container workloads.

Now reflect on this.

Do you know how to work with containers? Can you build one and can you run it locally?
If not, then it might be good to take a step back and do step 2 with the precondition instead!

#### 2. Create a lab environment
You now have an idea how the technology is used and it's time to just start experimenting. Setting up an environment
and getting familiar with tooling. In our case of Kubernetes, a good candidate could be to simply create a local cluster 
using for example `minikube`, `k3s`, `microk8s` or `kind`. Then being able to connect with it using `kubectl`.

Having a lab where you can try things out is golden! With Kind setup and interacting with your cluster you can take it
further.

#### 3. Create something simple
You have downloaded the tool you're exploring and you can now start creating something on your own.

If we continue on the Kubernetes example, this could be to deploy a container as a service without anything extra. Build
a very small http server which just returns the classic "Hello World!". When deployed, port forward to it and curl it to
see the response.

While you're doing this, you of course do research along the way. Using AI for understanding these parts is a great
addition to your toolbox.
