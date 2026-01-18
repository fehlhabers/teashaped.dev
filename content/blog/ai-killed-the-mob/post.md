---
title:  "How AI killed mob programming"
date:   2026-01-11
draft:  true
authors:
  - name: Kaj Fehlhaber
tags:
  - ai
  - mob-programming
---
I have been a great proponent of remote mob programming for quite a while now, creating collaborative cultures in
multiple new teams. Using git handovers and having timers to discuss how we solve different problems, keeping focus on
one thing. This has been vital for getting a team up an running, especially with a diverse team of different experience
levels. Not only for the knowledge spread, but also building culture.

However, times change. I see myself slipping away more and more from the regular mob sessions. I now have mob sessions with
AI agents instead.

## The new mob
What was previously a session where the goal might be to create a functionality and everyone getting familiar with the
details of the solution has now moving more and more towards handling higher level goals and to keep agents going in a
common direction. 

I no longer conduct mob sessions with the goal of refactoring code.

This is nothing that I have done by any specific stance or rule, but it rather has just naturally become this way. I
feel that it's hard to be able to manage multiple agents at the same time that I discuss with team mates. It's just too
much discussions going in.

## The new details
I start to care less and less if the code generated is exactly according to what my style of code is. What is important
is that the code does what it should. Refactoring is done at such speed by AI that it's no longer a large endeavour to
do so.

The hard part is to move from the habit of reviewing every line of code with the ambition of keeping the AI produced
code in the exact same style as that of what a human would write in the team.

## Swarming
What I think has become even more important is to *swarm* between team members to discuss architecture and the direction.

I mostly work with infrastructure and platform related tasks. This means that there is not just one big code base, but
rather many smaller pieces that combined becomes the platform. CLI for users to interact with platform backed services,
Helm charts for users and of course deployment of Kubernetes clusters with select operators, security tooling and
networking.

## The Team Differences

## Team Culture
I'm a bit torn in what I think of the development and how to place myself in the current landscape. Mob- and pair
programming have been my go to when starting new teams and to create a collaborative environment. There have been more
and less productive sessions, but I have always had the feeling that I can be away from work and someone else in the
team knows what needs to be done if something happens.

