---
layout: post
title:  "When managed services stop being fun" 
date:   2024-05-05
authors:
  - name: Kaj Fehlhaber
tags: 
  - cloud native
  - microservices
  - azure
  - postgres
---
![Cover Image](cover.jpg)
*Let me tell you a story about a team were everything was going really smoothly...*

The service was built up as an event based system with micro services which had adopted hexagonal architecture and
new features were easy to add.
The stateless apps could run nicely in Kubernetes while Azure Event Hub was leveraged as an event streaming
service. Azure Cosmos DB stood for the persistence.

*Life was good.*

Open api equivalents were used to integrate with the services instead of the native ones (Kafka API for events and MongoDB
API for document database) in order to make local development a joy. And who knows? Maybe it would come in handy at some
time?

## ⚠️ Trouble ahead
Using managed services aren't always all good. It usually comes with a price, and the question is if you are willing to
pay that price when the time comes? Let's look at what happened next...

### 6️⃣ The number of the tier
One day another feature was about to be added which required another event to be added to the
broker. The Terraform module was updated to add the new topic, role assignments prepared... A quick
IaC update which would allow for a quick addition of the additional service.

`terraform apply` rolls out in the pipeline... and a 🔴 appears!

`maximum amount of topics reached` it tells us. 🤯 But we have only **10** topics!

After having reviewed the [Azure Event Hub tier details](https://learn.microsoft.com/en-us/azure/event-hubs/compare-tiers), far down in the columns, it becomes apparent
that you would need to go the next pricing tier to go above 10 topics.

Next tier: **Premium**

According to the [pricing calculator](https://azure.microsoft.com/en-us/pricing/details/event-hubs/), this would move our service from consuming about $10-20 to $800 per month!

When serving a lot of traffic this might still be reasonable, but what about having a development or staging environment?
There is likely very little traffic, but yet you would still want to have the same setup in your staging as your prod to
verify that the IaC is setup correctly.
This makes this route a troublesome one, since the whole point with cloud is to be able to start small and scale.

#### What about AWS?
Maybe we were naive to believe that it would be possible to use Kafka compatible services which would not set us back
several hundred $/month just starting out, but AWS clearly also is very expensive just to get started according to the
[MSK pricing page](https://aws.amazon.com/msk/pricing/). Wasn't the point with serverless that it should be "pay per use"?

### 🔧 The "compatible" API
At the same time, features are being built which led to the [aggregate functions](https://www.mongodb.com/docs/manual/aggregation/) to be used in the MongoDB API when
fetching data* .
Local testing was done using [testcontainers](testcontainers.com) and things looked good. Bam! 🚀 Deployed the changes in
local [kind](kind.sigs.k8s.io) cluster and system tests are all green! 🙌

![Trying to make Cosmos MongoDB compatible](mongo_azure.jpg)
Time to merge the PR to build & deploy the updated app! 😃

System tests go 🔴... 😱

🤔 Head-scratching begins, but quite quickly the realization comes to us that Azure Cosmos DB for MongoDB API isn't really
MongoDB API compatible when you start wandering outside of the really vanilla queries.
It was simply not able to execute the query properly, but instead returned an empty result. What other differences are
there in behavior that we hadn't found yet?

**The whole solution screamed relational database, which was later used 🙏🐘*

## ➡️ What directions to take?
Having worked multiple years with AWS and Azure, there are many good products out there - but they often come with
caveats which teams either just throw money at to solve the issue or require a science degree just to optimize the usage
to make it cost efficient.

I'm more and more leaning towards options which include portable solutions instead of managed solutions - at least for
organizations of a certain size who can handle the overhead.
Nowadays, operators can be used extensively in Kubernetes to handle databases in higher degree and keeping to a
streamlined solution can build up expertise within the organization in that technology.

### 📚 Database
Postgres is an excellent example of a tool which has existed for a long time now and which can be used for most use
cases which services can have. There needs to be a hell of a lot of write requests before there is any need of more
advanced sharding. It also supports both relational as well as [document store](https://www.postgresql.org/docs/16/datatype-json.html).
A Kubernetes deployed Postgres can scale to many instances which can serve read traffic and can be vertically scaled to
handle many thousands of write operations per second.

Maybe a good fit for your team as well?

![Postgres & Kubernetes](pg_k8s.png)

### 📨 Messaging/events
Kafka is the de facto standard today when it comes to handling events between services. But there is also a very
interesting competition out there, which has been around for quite some time but which has gained more traction recently
with the introduction of their new feature "JetStreams". I'm talking about [N.A.T.S](nats.io), which is an incubating
project within the [CNCF](landscape.cncf.io) landscape.

Like most good apps, it's written in [Go](go.dev) 😉 and can be deployed as a cluster, where the application is just a few
MB. This is definitely worth taking an extra look at.

Did I mention you can use it as a key-value store as well? Check it out! It has a bunch of features bundled in a very
small package which all have the necessary ingredients to improve development.

![NATS](nats.png)

## 💭 Final Thoughts
Managed services definitely have their place in a lot of use cases, but as with many others, the pendulum has reached the
limit for me and it's currently going back more to self-managed services to a higher degree.
Don't be afraid to use them, but **do** have a strategy for how you can switch provider and how you are able to test
applications locally in a good way. 
If keeping the technologies that hold persistent data to a minimum, you are more likely too build up competence around
these technologies so that you can handle disaster recovery in the same (or better) way as with managed services.

I just brushed lightly on the messaging and database topics, but it's worth diving into more deeply at some other time.
<br>
<br>

**Kaj Fehlhaber**
