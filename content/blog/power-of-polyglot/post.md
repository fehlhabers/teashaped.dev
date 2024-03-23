---
layout: post
title:  "The usefulness of polyglot services" 
date:   2024-03-23
authors:
  - name: Kaj Fehlhaber
tags: 
  - polyglot
  - cloud native
  - java
  - microservices
  - golang
---
![Cover image](cover.png)
We we're just about to start with what we had thought to be a very simple converter... You know:
When for example receiving a protobuf or json of some sort which is then transformed into a domain
object for further handling.

Only this time it was a little bit different...

## EXI? What the #!*$ is that?
We knew that we would need to decode and also encode a format known as `exi` and that it is a format
[defined by _W3S_](https://www.w3.org/XML/EXI/). It is a binary format of xml which reduces the 
payload significantly. A similar use like `protobuf` or `avro`. Never heard of it before, but it was used by an ISO
standard we would need to implement. How hard would that be? 

On further investigation there were only a handful of implementations of the encoding **and they
were all in either Java or C**. The implementations were also mostly done by few contributors and
were of academic nature.

I was almost shocked by the realization that the decoding of the payload would actually be much more than
a simple:

```go
err := proto.UnMarshal(payload, &message)
```

It was clear that `exi` must've been defined just before protobuf and other binary formats which
became more popular and the ISO standard must've been set just at that time as well.

## Alternatives
My usual workflow when invesigating a new topic is like what many others do: Search for what others
have done, use ChatGPT and then use the input to get some bearings on which routes would be best to
take.

The result was very much what we had dreaded... There is really not much use out there. The
repositories of the different implementations have mostly not been touched for several years and
the only implementation which seemed remotely maintained looked to be [`exificent`](https://github.com/exificent/exificent)
, which is a java implementation.

Our main stack is based on `Golang` so we used that as base when looking at alternatives. We came up
with the following options:

- Use a C library and interact with it using CGO
- Interact with the Java library using JNI
- Build the functionality of encoding/decoding in a separate pod
- Create an own implementation in Go

(Not all necesarily feasible)

### Cognitive load - considerations
Our team is quite capable in several different languages, but it's not sure that it looks the same
way in a couple of years. How do you keep the service easy to understand for newcomers and keep it
easy to maintain?

My recommendation would be: **Keep to a common pattern**

This is how we build everything: 
- CI/CD is handled the same for all apps.
- The architecture within apps is similar so there are no surprises
- Secret-handling, DB, event streaming, etc

So the question is which of the options would add the least overhead without sacrificing too much
performance?

## A new service is born!
As the average reader might have figured out from the title, we did move forward with the approach
with creating a separate pod for this specific use!

This is one of the major benefits when it comes to building in a micro service architecture. Pods
can communicate with each other over **network** which can be any sort of API! When combining this
idea with a standard interface such as gRPC or REST it becomes very easy to build pods in different
languages when the need arises.

*Another use case could be that a service would need AI capabilities which is only implemented in
Python.*

### The implementation
Since the actual enrichment of the `exi` payload was very minimal, the service could focus on only
enriching an incoming `exi` and return the enriched payload right away. The rest of the
functionality could be done in our main service which was written like our other micro services.

We could now build a very small java app which basically consisted of `quarkus` and `exificent`
where a REST endpoint was exposed to handle the enrichment in about 
20 lines of code.

Our idea was that you would not need to be a java expert to know what to change if needed. Strip
everyting down to a minimum so only the real basics are there. Plain `maven` (Holy shit, that is
such a pain now when having been away from Java for about a year!) to build an uber-jar.

You should also be able to easily handle updates. Dependabot would help with constantly creating PRs
and the app would be so minimal that it would be very easy to upgrade without any risk of it
breaking.

### Containerization and cluster setup
The container could be built basically like every other of our apps: A builder stage which builds
the binary (in this case an uber-jar) and then a minimal image which can run it.
The "minimal" is of course much bigger in a Java context, since it requires a `jre` to run.

We did consider to build pods where the java app would run in the same pod but in another container.
This would make sure that the network overhead would be minimal since they would always run on the
same node, but would also mean that the scaling would likely be much more sluggish since they would
need to scale differently and startup of a Java app is much slower.

By setting node affinity of the pods we can try to make the pods be close to each other. Since the
service will not be needing any real-time feedback, the latency would be negligable.

## Conclusion
As a mob team of three we were able to...
- Determine a viable solution 
- Build and containerize the app
- Altering CI/CD for security scans for Java
- Setting up Helm Charts
- Deploy to environments

... within less than a day.

Compare that with trying to squeeze in the solution to your favorite language through alternative
ways. Researching how to do this and also having a special way of doing it which might be forgotten
a year later.

So next time you tackle a problem which might be better done in another language - just build
another app and communicate with the app using standard interfaces (which are widely adopted...
maybe not `exi`).

Hope this helped someone out there!
