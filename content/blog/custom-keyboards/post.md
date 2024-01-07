---
title:  "A dive into custom keyboards"
date:   2024-01-04
authors:
  - name: Kaj Fehlhaber
tags:
  - keyboards
---
I have never understood the custom keyboard community... Why would you spend so much time and effort to have a special keyboard?
Wouldn't it just make it harder when getting to a regular keyboard and make yourself tied up?

**...Only to now find myself in this maze of split keyboards with custom bindings and what not!**

Let me share why I did this 180 and how I took the plunge and also (started to) get productive with my first custom keyboard.
This is somewhat of a different blog from what I have usually done, but thought it could be a good help for anyone getting started.

# Reasoning on starting
After summer I started exploring the notion of ditching the Swedish layout since a lot of the symbols that are used in programming were placed on awkward locations.
It would make more sense to use the US layout instead for this reason. So begins my journey...
Obviously, using a US layout means I need to handle åäö when typing Swedish somehow.

Introducing xkb and creating custom layouts!

At this point I had started my journey towards **Vim** (or rather **Neovim**). While I felt that the concept of ditching the mouse in
favor of moving around with powerful bindings, there was a feeling that I had trouble touch-typing characters that were farther from homerow.
Didn't think too much about it, but did get a cheap 60% keyboard from Amazon to be able to focus more on the keys near homerow.

I have the fortune to work with a bunch of great individuals where we constantly share experiences and ideas. When we stumbled upon a problem of some sort during the day, the chance is high that someone researched it and came up with a solution proposal or maybe created an open source tool which for the general problem at hand.

**What does this have to do with anything about keyboards?** 🤔

I'll tell you why...

**Because our team nurtures a culture of constantly improving each other and what we do!** As a part of that, we spur each other in exploring areas just like custom keyboards, since it is an area which has a good chance of having a positive impact.

During one of our discussions where a new member shared some about his new Glove80, the question about price of course came up. I hadn't been willing to make any particular investment before, which I stated. One of the replies sounded something like this:

***"I'm using my keyboard 8h a day. Investing in a keyboard is a no-brainer for me!"***

That evening I made the decision that it was time to at leart consider exploring custom keyboards.

# What options are out there?
I have quite small hands and a reason for even looking at custom keyboards was that I saw a much higher likelihood of being successful
at touch-typing with a smaller form factor keyboard.

There are sooo many variants out there, but that's kind of implied since they are **custom** keyboards. I identified some interesting groups to look more into:

### 60% form factor
The first path I went, which I still believe is a good first step with out too much training need. You get rid of fn, nav & num cluster and have the main keys left.

### Ortholinear
The way fingers need to move on a regular keyboard is quite straining, since keys are placed diagonally. If you try to touch-type, you will notice this quite soon. Ortholinear keyboards place keys in a grid which makes reaching them more natural.

### Split keyboards
I had previously dismissed split keyboards since it didn't work well with how I type(d), but since part of the journey was to learn touch-typing properly it was back on the table.

These come in a large variety, but I was aiming at one with low profile, small form factor and an aggressive column-stagger. Column-stagger means that the keys are staggered in columns so that every finger can get their keys at a distance better suited for the length of the finger. An improvement to the ortholinear placement, really.

The split setup made it possible for all keys I needed to placed in a way which made them easy to reach, so this was the obvious path for me.

I didn't really bother with the bigger form factor, since it was one of my drivers to be able to use a keyboard with few keys.

# Keyboards and open source
Now that I had an idea on what to look further into, it was time to narrow down the options.

I quickly found out that there are a bunch of open source projects out there with designs - just like with any other software!
My interest soon started to move towards keyboards such as [Corne](https://github.com/foostan/crkbd), [Piantor](https://github.com/beekeeb/piantor) & [Sweep](https://github.com/davidphilipbarr/Sweep). These are minimal keyboards which works with layers rather than many buttons.
Of course, these differ vastly from a regular keyboard and require that the keys are mapped out to what suits my need!

Enter [**QMK**](https://docs.qmk.fm/#/) & [**ZMK**](https://zmk.dev/)!

These are excellent tools for flashing your keyboard with exactly the behavior you want and open up a completely new world when it comes
to how to use your keyboard. You can for example set different behavior on tap vs hold - this enables you to use your homerow for shift, ctrl, etc as well! Or the feature to use CAPS WORD, which turns on CAPS for a word, then turns it off!

You can configure your completely own keyboard or you can use countless of configurations which the community has shared.

## Variations
The open source variants
