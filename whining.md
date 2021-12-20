---
title: "Stop Whining about Rust Hype - A Pro-Rust Rant"
description: "A rant by an annoying Rust advocate"
layout: post
date: 2021-12-20T00:00:00Z
---

If you don't like this article: I was very careful when writing this piece to
avoid qualifying or couching anything I say. This article is catharsis borne of
my the daily frustration I feel when I know there's a better way, but I'm not
empowered by my job to take it. There's even a handy list of at the bottom of
all the things I removed.

# Stop Whining About Rust Hype

Every discussion involving Rust ends up with an interminable thread complaining
Rust "hype". It's "overhyped". How much you hate hearing about Rust everywhere.
Rust, Rust, Rust! Won't all these "fanbois" just go away and stop talking about
Rust?

Do you remember when Java was the hot new thing? No, probably not. You're
probably too young. Well, back in the day, there were trade magazines written
on actual dead paper, but about computers (weird, I know). Those magazines were
*full* of articles about Java, its promise, and the problems it would solve.
The Internet (back when it was capitalized) wasn't yet a hate-powered echo
chamber, so instant knee-jerk reaction boiler rooms weren't as much of a thing
but the complaints were the same. Too many articles about this unproven
technology. Nobody used it, so nobody knew it. Nobody knew it, so it couldn't
be adopted. Virtual machine runtimes had been tried before, C and COBOL were
good enough, I'm tired of hearing about it, and on and on.

We all know how that ended. Java lived up to the hype, and ate the industry for
20 years. Let's talk about why you need to stop whining about the hype.

## Why Is There So Much Hype?

Before Rust came along, there was no point to repeatedly highlighting problems,
because there were no real solutions. Everyone knows buffer overflows are a
problem, and languages like Java help. Everyone knows that having to write your
own data structures sucks, and languages like Python help. Discussing pain
points in terms of entire _categories_ of problems (like "ease of composition"
and "memory safety") hadn't entered the popular zeitgeist because it wasn't
fruitful to do so unless you were designing a programming language. Security,
of course, has been a well-understood category of problems for decades, but
solving it either required tradeoffs in performance and maintainability
(Python, Ruby, Erlang) or not actually solving it (Java, JavaScript, PHP).

These problems, entire categories of problem sets, exist as "background
radiation". Everyone complains about them all the time, but there's no solution
to all of them. With Rust, you're hearing about a technology that _can_ solve
all of them. Now instead of a many-to-many problem-to-solution mapping, it's
many-to-one. Now people talk about solving your problem, _and_ your problem
_category_, and _other_ entire categories of problems to boot! This makes it
feel like Rust is everywhere, all at once, because it's relevant to everything
we do.

## Stop Lying to Yourself

It's a hallmark of techie-types, of "engineers", that you're adept at assessing
systems dispassionately. You can put aside the "hype" and consider solutions on
their merits. You can decide based on fact, and not emotion. Resistance even to
discussion of Rust because of "hype" belies the truth of that. It's ad hominem
in the extreme.

Whining about Rust hype is damaging and insulting because, in doing so, you're
accusing me of acting in bad faith. I'm not a shill. I don't get paid by the
Rust Foundation to convince you to buy Rust Enterprise. I'm also not an idiot.
I've been programming for 30 years. I've successfully done big refactors in
languages that don't have type safety. I've written fast services in languages
that incur GC overhead. I've written tight code in languages that don't enforce
good memory hygiene. I've done it on tiny micros, and I've done it on
distributed multi-core clusters. I've been there. I've fucking done that. And I
know a good thing when I see it.

I'm bringing Rust up in your thread because it's relevant, and probably solves
a problem you've got (even if you're inured to it). I don't owe you an
exhaustive pros-vs-cons article any time I write anything. If you choose to
disagree with me, disagree with me on the merits of what I've said, and please
respond to me in good faith, as I say what I say in good faith.

## Stop Tone Policing

But most of all, stop tone policing. That's exactly what you're doing when you
complain about "too many" people "overhyping" Rust. Complaining about that is
complaining that people are saying anything at all, or saying it in the wrong
way. If you're tired of hearing about it, well, I'm tired of hearing about
half-solutions and solved problems causing trouble yet another time.

I'm writing because I have something to say, and that's the point. You're
welcome not to read it; I won't be insulted. You don't owe me your attention.
But I also don't owe it to you to cater to your delicate sensitivities and
apologize for every other excited person who also writes about Rust. Most of
all, I'm not going to stop advocating for something I believe will materially
improve the industry (and my job satisfaction).

## You're Threatened

Java was also a "fad" that would go away once the "hype" died down. And anyway,
"real" programmers didn't write Java, which I guess isn't a problem Rust has
because Rust is "hard". (It's not, by the way. It's not hard. It's truly not.)
Nobody was threatened by Java because the growth of the Internet (and the
industry) meant nobody was competing for *your* job, only *new* jobs. And
because snobbish attitudes kept a cynical separation between the "real"
programmers and the unwashed masses.

An ability to compensate for the shitty state of technology that exists today
isn't some kind of a competitive "moat". Nobody thinks you're smarter or better
because you can remember all the pitfalls. Learning all the tricks and caveats
to avoid problems is the equivalent of learning alchemical potion recipes.
Someone who *could* learn these things but doesn't have to spend the energy on
it is going to eat your lunch. Some company that saves money by not spending as
much on debugging or refactoring, and that avoids paying for security fire
drills, and saves money on hardware costs by running close to the metal is
going to eat your lunch. I can write Rust as quickly as I can write Python, and
other people can too. Time-to-market matters, and the gap between Rust and
scripting languages is closing fast. Soon your solution won't be faster to
market, and will be more expensive to maintain to boot. And someone's going to
eat your lunch while you complain.

There is a competitive advantage to knowing Rust right now. Hiring managers are
using it like a filter for the best people because knowing Rust helps you write
better code in other languages. In the near future, it's going to be table
stakes, and shooting all the messengers in the world won't change that.

## But What About...

A list of polite things you expect me to say, but I left out:

* Java failed in as many ways as it succeeded.
* There's a time and place, and you don't want to drive people away.
* Some people have known about categories of problems since the 60s and have
  tried solving them before, and failed.
* Maybe all the code I've written over my career sucks.
* Sufficiently skilled programmers can overcome or avoid the pitfalls of other
  languages.
* You can write bad code in any language.
* You can write insecure code in any language.
* I'm not talking about *you*, specifically, dear reader. The _general_ you.
* Rust doesn't solve every problem, of course, so I won't claim that it does.
* I have seen other good tech that isn't Rust.
* Rust is a big language so there's a lot to learn, and that's hard.
* It's hard to *measure* how much Rust improves things.
* Some of Rust's difficulties and problems can't be solved and never will.
* Working with shitty technology is a competitive advantage, it's just not a
  growth market.
* Maybe working with shitty technology *is* a growth market, because we keep
  making more.
* Maybe Rust is more shitty technology, and I just don't know it yet.
* The speed at which I write code is not actually impressive.
* Please, I'd prefer it, it would be better if, won't you, I'd like it, I'd
  appreciate it, but I can't tell you what to do.
* This article is whining too.
