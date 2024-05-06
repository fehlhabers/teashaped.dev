---
layout: post
title:  "Considerations with managed services" 
date:   2024-05-05
authors:
  - name: Kaj Fehlhaber
tags: 
  - cloud native
  - microservices
  - azure
  - postgres
---

Let me tell you a story about a team were everything was going really smoothly...

The service was built up as an event based system with micro services which had adopted hexagonal architecture and
new features were easy to add.
The stateless apps could run nicely in Kubernetes while Azure Event Hub was leveraged as an event streaming
service. Azure Cosmos DB stood for the persistence. Life was good.

Open api equivalents were used to integrate with the services instead of the native ones (Kafka
API for events and MongoDB API for document database) in order to make local development a joy.

## The number of the tier
One day another feature was about to be added which required another event to be added to the
broker. The Terraform module was updated to add the new topic, role assignments prepared... A quick
IaC update which would allow for a quick addition of the additional service.

`terraform apply` rolls out in the pipeline... and a 🔴 appears!

`maximum amount of topics reached` it tells us. 🤯 But we have only **10** topics!

After having reviewed the Azure Event Hub tier details, far down in the columns, it becomes apparent
that you would need to go the next pricing tier to go above 10 topics.

Next tier: **Premium**

This would move our service from consuming about $10-20/month to $800/month!!
When serving a lot of traffic this might still be reasonable
