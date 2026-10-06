---
layout: post
title: "Clutch Run: A Driving Game About Changing Gear"
date: 2026-10-06
categories: [cars]
permalink: /cars/clutch-run/
excerpt: "A small 3D driving game in the browser for practising manual gear changes on different terrain: climbs, descents, gravel and village streets. Plus a fun automatic mode for phones, where tilting the phone is the accelerator."
---

Most driving games either hide the gearbox or turn it into a button. In a real hatchback the gearbox is the whole lesson. You stall at the lights, you lug the engine in fourth up a hill, and you wonder why the car shudders.

**[Clutch Run](https://and.github.io/clutch-run/)** puts that lesson in a browser tab. The idea is to practise changing gear by hand on different terrain. You drive a small petrol hatchback round a hill loop with village streets, a steep climb, a gravel stretch and a long descent. Each one asks for a different gear: pull away in first, drop to second for the climb, hold third down the hill and let the engine brake. You change gear yourself, keep left, and stay off the other traffic.

For phones there is also a fun mode: an automatic car where you tilt the phone to accelerate and brake. It works on a laptop with the keyboard and on a phone held upright or sideways. There is nothing to install.

**[Play Clutch Run →](https://and.github.io/clutch-run/)**

<figure>
  <img src="{{ "/assets/images/clutch-run/desktop-start.jpg" | relative_url }}" alt="The Clutch Run start screen on a laptop: clutch mode, key layout, road, time and weather" loading="lazy">
  <figcaption>The start screen on a laptop. Pick the clutch mode, the keys, the road, the time and the weather.</figcaption>
</figure>

* Contents
{:toc}

## An engine you can stall

The car is a simulation, not an animation. The engine has a torque curve that peaks at 115 N·m around 4,200 rpm. It idles at 850 rpm, redlines at 6,300 and stalls below 420. The clutch slips through a bite zone, so letting it up too fast with too few revs kills the engine, just as it does in a real car.

The gear ratios match the car in [Reading the Rev Counter]({{ "/cars/rev-counter/" | relative_url }}), so the speeds feel familiar:

| Gear | Ratio | km/h at 3,000 rpm | km/h at the redline |
|---|---:|---:|---:|
| 1st | 3.55 | 23 | 49 |
| 2nd | 1.95 | 43 | 90 |
| 3rd | 1.28 | 65 | 137 |
| 4th | 0.97 | 86 | 180 |
| 5th | 0.78 | 107 | — |

*Final drive 4.07, 0.30 m wheel radius, 1,050 kg car. Speeds are worked out from the ratios.*

The latest version makes the car feel heavier on the road. Lifting off the accelerator brings real engine braking: about 0.9 m/s² in second gear at 50 km/h, against 0.3 in neutral. Rolling resistance grows on gravel and grass and in tight corners. The body pitches when you brake, leans in bends and picks up small bumps from the surface. The car no longer glides as if it were on ice.

## A gear lever under your thumb

![An H-pattern gear gate: 1, 3 and 5 on top, 2, 4 and R below, with the knob in neutral in the middle]({{ "/assets/images/clutch-run/gear-gate.svg" | relative_url }})

On a phone, the gears are an H-pattern gate that you drag through. It is laid out like most Indian hatchbacks: 1, 3 and 5 on top, 2, 4 and R below. The knob only moves along the slots. If a gear is refused (first gear at 80 km/h, or reverse while moving), the knob springs back to neutral with a grinding buzz.

<figure>
  <img src="{{ "/assets/images/clutch-run/phone-manual.jpg" | relative_url }}" alt="Clutch Run on a phone held sideways: the gear gate top left with the knob in 3rd, the rev counter and speedometer, and brake and accelerator pedals on the right" loading="lazy">
  <figcaption>A phone held sideways in manual mode. Third gear, revs in the green, a car coming the other way.</figcaption>
</figure>

Sideways, the gate sits in a top corner of the road view, so your eyes stay on the road while you change gear. Upright, it sits at the foot of the view.

You steer by turning the phone like a wheel. The accelerator and brake are analog: press higher up a pedal to press harder. The start screen asks whether you sit on the right (gears on the left, as in India and the UK) or on the left, and mirrors the controls.

## Automatic mode: the phone is the pedal

Manual is the lesson; automatic is the fun mode for phones. **Automatic** mode swaps the lever for a P R N D selector that you drag like a real one, and changes gear by itself. The drive starts in D, so you can go at once.

Here the phone's angle does the pedal work. Whatever angle you hold the phone at when the drive starts becomes *rest*, about 35° for most people. Tilt the top edge back to accelerate and tip it forward to brake:

| Phone angle (rest at 35°) | What happens |
|---|---|
| Above 60° | Full accelerator |
| 40° to 60° | Accelerator, harder the further you tilt |
| 30° to 40° | Coast |
| 10° to 30° | Brake, harder the further you tip |
| Below 10° | Full brake |

P and R need the car stopped, as in a real automatic. Drag the knob to R while rolling and it springs back to D.

<figure>
  <img src="{{ "/assets/images/clutch-run/phone-auto.jpg" | relative_url }}" alt="Clutch Run on a phone held upright at night in automatic mode: a starry sky over the village, D1 on the dashboard and the P R N D selector set to D" loading="lazy" style="max-width:360px;width:100%">
  <figcaption>Automatic mode on a phone held upright, at night. Tilt back to go; the selector is in D.</figcaption>
</figure>

## Endless roads, weather and the clock

Besides the hill loop with lap times, there is an **endless road**. The game builds it ahead of you as you drive, with hills, bends, gravel, villages and traffic, and clears it away behind you.

The sky follows your own clock by default: dark at night, orange at dusk. Rain makes the road slippery and fog closes the view. Headlights are a key or a button away.

<figure>
  <img src="{{ "/assets/images/clutch-run/desktop-rain.jpg" | relative_url }}" alt="Clutch Run on a laptop in the rain at dusk: the hatchback in third gear at 28 km/h on a wet village road" loading="lazy">
  <figcaption>Evening rain on the hill loop, third gear on the keyboard.</figcaption>
</figure>

## How it is built

- **No build step.** Plain JavaScript modules and [three.js](https://threejs.org), served as static files from GitHub Pages. About 2,200 lines in all.
- **No model files.** The terrain, road, village, trees and cars are all made in code.
- **No sound files.** The engine note, tyres, crashes and gear grinds are synthesised live with the Web Audio API, following the engine's real rpm.
- **Phone sensors.** Steering and the automatic's pedals read gravity from the phone's motion sensor. On Android, each gear change clicks with a short vibration.

The first playable version went up on 30 September 2026, and version 22 followed six days later. Each release shows its number on the start screen, so you can tell whether your phone has the latest one.

## Play it

[and.github.io/clutch-run](https://and.github.io/clutch-run/). A phone gives the full experience. On a computer, add `?touch` to the address to try the phone controls. The source is [on GitHub](https://github.com/and/clutch-run).
