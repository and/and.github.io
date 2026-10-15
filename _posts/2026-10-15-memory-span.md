---
layout: post
title: "memory.span: How Many Things Can You Hold in Your Head?"
date: 2026-10-15 12:00:00 +0530
categories: [dsa, games]
permalink: /games/memory-span/
excerpt: "A small browser game that measures your working memory: digits forwards, digits backwards, and a pointers mode that asks you to track variables as they are copied and changed. Practise offline, or play Live on a public leaderboard."
---

[memory.span](https://and.github.io/memory/) is a small game that finds how many things you
can hold in your head at once. Digits flash one at a time, and you type them back. Each right
answer adds one more. The longest sequence you get right is your span.

![The memory.span start screen: three game cards (Digits, Reverse and Pointers), the Practice and Live switch, and a demo of the Digits game typing 5 and 2]({{ "/assets/images/memory-span/start.png" | relative_url }})

## Why I made it

While working through [zipper lists]({{ "/dsa/zipper-lists/" | relative_url }}), I kept losing
track of which pointer pointed where. `current_1`, `current_2`, `next_1`, `tail`: after three
or four of them I would forget one and the loop would go wrong. I wondered how many I could
actually hold, and whether I could train it.

## Three games

- **Digits.** Digits flash one per second; type them back in order. This copies the standard
  digit-span task. Most adults land between 5 and 8.
- **Reverse.** The same, but type them backwards. It's harder: most adults land between 4
  and 6.
- **Pointers.** Lines like `a = 4`, `b = a`, `a = 9` appear one at a time, then you give the
  final value of every variable (here, a is 9 and b is 4). This is the one I made the game
  for, because it is the closest to tracing pointers in a linked-list loop. There are no
  published norms for it, so the leaderboard is the only yardstick.

You have three lives per game. A miss costs one, and right answers in a row build a streak
that multiplies your points. The start screen plays a short demo of the game you pick, so you
can see what to do before you press start.

## Practice or Live

A switch next to the title picks how you play.

- **Practice** runs entirely in your browser. Nothing is sent anywhere, and a small graph under
  each game tracks your progress.
- **Live** puts your games on a public leaderboard, with a tab per game and a **Total** tab
  that adds up each player's best in every game. The first time you play Live you get a
  random nickname, such as "Swift Otter 42", which you can keep or change.

## Keeping the leaderboard honest

A leaderboard is only fun if the scores are real. If the browser made the rounds and checked
the answers, anyone could open the developer tools and post a span of 40. So a small
[Cloudflare Worker](https://workers.cloudflare.com) does both jobs:

- The server creates every round and keeps the answer. The browser only gets the sequence it
  has to show.
- An answer that arrives before the sequence could have finished showing is refused.
- At most 60 games per player per hour.
- Nicknames must be unique, are filtered for offensive words, and can't be near-copies of
  another player's name. "Swift Otter" and "swift_0tter" count as the same name.

It can't stop a bot that watches the screen and types the digits back, and no memory game in
a browser can. But it does mean nobody can just make up a score.

## A few small things

- **End a game early.** Press Esc or click **End game**. In Live, the game counts with the
  score you had so far, so quitting can never raise a score.
- **Light and dark**, sound on or off, and a still picture instead of the demos if your system
  asks for reduced motion.
- **Not a clinical test.** Digit span is a real measure of working memory, but this is a game,
  and scores move by about one level from day to day. Sleep and coffee count.

## What's next

Once each game has about 30 Live players, your progress panel will say how you compare, for
example "Your best span 7 beats 68% of players". After that, an optional age bracket, so you
can compare with people your own age.

## Try it

[Play memory.span](https://and.github.io/memory/). It's free, there's nothing to install, and
[the source is on GitHub](https://github.com/and/memory).
