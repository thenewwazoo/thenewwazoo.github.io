---
title: On Abandoning my RATGDO Native HomeKit Users
description: In which I explain why I am leaving wonderful people high and dry
layout: default
date: 2024-02-23T00:00:00Z
---

tl;dr: I'm retiring from the ESP8266 Native HomeKit RATGDO codebase, and recommend that it not be
used. I'm going to move forward with an ESP32 version to scratch my own itch, and hopefully it'll be
useful to others. I'm genuinely sorry to, and thankful to, everyone who used the code I wrote. Skip
to [What's Next?](#whats-next) if you are looking for an alternate recommendation.

Let me open this by stating who I am: I'm just some guy. I don't sell the RATGDO hardware (that's
Paul), and I paid for my first one (Paul sent me a second for free to develop with, which was
very kind). I _would_ really like to do embedded work for a living, so if you'd like to hire me, you
should get in touch. But aside from that, I'm just a person who had an itch, and the time and skill
to try and scratch it myself.

## An apology

This story is a hard one to tell because it involves speaking negatively about lots of awesome
contributions to the commons by well-intentioned people (including myself!). The world has been
improved and enriched by the existence of Arduino, despite it (IMO) not being a good basis for
"production" hardware. Likewise with the ESP8266-compatible HomeKit implementation for the Arduino
platform, which is no longer maintained and has significant bugs. And so on, up the whole stack.

I feel deep gratitude for all of these things, but after months of hobby-time work I wasn't able to
build a product using those components that was of the quality I wanted to provide for the people
who were using my code. That's frustrating. Is it a failure? It's hard not to feel that I've failed.
Perhaps with enough dogged determination I could have fixed it all.

Or maybe I could spend my limited free time in ways that I find more satisfying. Is _that_ a failure?

So I've now got probably hundreds of users who have been dealing with a buggy, crashy firmware in
the hopes that some day I'll deliver improved quality. And I won't. Not with the hardware they've
bought.

## What's next?

For those users who have already purchased a RATGDO based on the ESP8266 (that is all of them),
there are a handful of options. I'm assuming that anyone interested in this post wants their garage
door to be controllable via HomeKit. In that case, the best option is probably to use Homebridge and
the homebridge-ratgdo plugin, along with the MQTT firmware. I like Homebridge, and run it quite
successfully on a Raspberry Pi 4.

Another option is to use Home Assistant and the ESPHome firmware, along with the HomeKit
integration. The ESPHome firmware is, overall, nicer code. I have personally had a bad time with
Home Assistant but others have had great success.

There is no RATGDO on the market today that has an ESP32 on it. Certain older RATGDOs have modules
that are connected via headers (versus having ESP12 modules soldered directly to the carrier board).
I happen to have bought one of those older modules, and also have some Wemos D1 Mini-compatible
ESP32 boards on hand. So I'll be using those to port the existing code to the ESP32, and releasing
it.

The last option is to keep on truckin' with the native HomeKit firmware I wrote. There is a small
community of contributors who are interested in keeping it going, most especially [jgstroud], who
disagrees with me that the task is hopeless. They are still adding features and trying to fix the
failures.

## How did we get here?

When Chamberlain decided to block access to their APIs without engaging with the community of
developers who'd worked to build upon it, I went looking for alternatives. There were a small number
of commercial options, and one option that appeared much more in the hacker spirit: the RATGDO. At
the time, there were two firmwares available: one using MQTT, and another built upon ESPHome. There
was quite conspicuously not a HomeKit option.

I've always had an interest in HomeKit. I'm fully bought into the Apple ecosystem, and like Charlie
Brown trying to kick the football, I've always hoped that HomeKit might one day live up to its
potential. Instead the ADK remains a bit of a pig, the spec is (partly) locked away, and the Home
app is one of the best examples of Apple's institutional dysfunction leaking out for the world to
see. Multiple little birdies have told me that the entire HomeKit team is overworked, burned-out,
and under-funded, and _it shows_.

But I digress.

I'd used HomeSpan before to build a logic board replacement for my kid's window fan, so I thought,
"I know, I'll write a native HomeKit firmware!" And because every good idea deserves to be brought
to market as quickly as possible, I grabbed the MQTT firmware (based on Arduino) and an
off-the-shelf port of the HomeKit SDK, and got to hacking. But not with HomeSpan, because that
doesn't support the ESP8266.

As I made progress, received user trouble reports, and worked with other contributors to the
codebase, it became clear that there were serious problems with the platform I'd chosen. The
Arduino's one-big-loop style prevented me from making certain features work elegantly, and the
unmaintained HomeKit SDK port I was using had bugs that caused poor user experience. Devices were
crashing, pairings were getting lost, WiFi performance was unreliable, and many more problems.

After discussing with the other contributors, we concluded that we could either take ownership of
all the code's dependencies ourselves and fix them, or try moving away from Arduino entirely. I
decided that I'd build an experimental fork based on the Espressif HomeKit SDK, which ostensibly
supports the ESP8266. It would be a proof-of-concept. Also I don't get to do clean-sheet rewrites in
my day job, so that'd be fun.

After several weeks, I had a set of components that implement all of the features the RATGDO
requires. And they all worked, in isolation. But turning them all on at the same time led to
out-of-memory crashes.

Let me digress again for a moment: It's a totally reasonable engineering decision to use a heap in a
network-connected embedded device in order to handle burst-y load. When there is no way to know, in
advance, how many simultaneous connections you'll need to service, or how many queries you'll get in
short order, and so on, the usual approach of statically, and rigidly, allocating buffers in advance
can end up starving every individual part of the system, and gaining you nothing. Using a heap
reduces determinism but the increase in flexibility can make the difference between your device
living and dying.

The Espressif SDK design clearly makes that tradeoff, but it can't solve the fact that the ESP8266
is simply RAM-limited when running HomeKit. I, and others, have spent many hours now sizing buffers,
and shaving stacks, and trying to find the right combination of RTOS task setup that will permit the
whole system to work reliably and it has not succeeded. Indeed, in doing so I have become convinced
that many of the otherwise-inexplicable problems we're seeing in the Arduino-based code are caused
by hidden out-of-memory errors causing corruption.

Perhaps with enough deep knowledge of HomeKit and the Espressif SDK's implementation details and
some rigorous engineering discipline I could characterize the system in sufficient detail that I
could build a reliable system. Or I could do something more fun.

Is that a failure?

## A technical retrospective

Okay, here you'll find a cop-out. The reasons I'm convinced that it's hopeless to fit the RATGDO
firmware on an ESP8266 are actually pretty interesting, but I've been sitting on this post for a
while, and fielding questions about support and the future of the device. I'd really like to publish
this so I can point interested users to it. As such, I'm going to write it up Real Soon Now.

## Adiós

So that's where I am.

[jgstroud]: https://github.com/jgstroud
