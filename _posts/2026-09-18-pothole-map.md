---
layout: post
title: "Finding Potholes with a Phone"
date: 2026-09-18 20:00:19 +0530
categories: [utilities]
permalink: /utilities/pothole-map/
excerpt: "A phone app that notices potholes by feel while you drive, and records where each one was and how hard it hit. It is a private build I use myself, and it has only had its first day on the road."
---

Pothole Map is an app that finds potholes while you drive, without you doing anything. The phone feels the jolt, notes the place and how hard the hit was, and saves it. This is a private build that I use myself, and it is not on any app store.

## How it works

1. **It notices you are in a vehicle** and starts recording by itself. You can also start it by hand.
2. **It watches the phone's motion sensors and GPS** for sudden up-and-down jolts.
3. **It saves each jolt** with its location, the speed, and an impact score.

The app measures the jolt straight up and down, so it doesn't matter how the phone is lying in the car. The Android and iOS apps follow the same written rules, so their reports can be combined later.

The plan is a public map: when several phones report a jolt in the same place, it counts as a confirmed pothole.

<div style="display:flex;gap:16px;flex-wrap:wrap;justify-content:center;margin:24px 0">
  <figure style="margin:0;flex:1 1 220px;max-width:300px">
    <img src="{{ "/assets/images/pothole-map/app-home.jpg" | relative_url }}" alt="The Pothole Map home screen on Android: 549 events saved, a switch to detect trips automatically, buttons to start recording, export and clear, battery use figures, and a list of recent detections with impact scores and speeds" loading="lazy">
    <figcaption>The home screen, with the battery log and recent detections.</figcaption>
  </figure>
  <figure style="margin:0;flex:1 1 220px;max-width:300px">
    <img src="{{ "/assets/images/pothole-map/app-submissions.jpg" | relative_url }}" alt="The Your submissions screen: 549 reports over 1 day, 40 severe, and a list of reports marked Severe, Moderate or Minor with their speed and impact score" loading="lazy">
    <figcaption>Every report, marked minor, moderate or severe. I hid the locations.</figcaption>
  </figure>
</div>

## What the first day showed

On 22 September 2026, the app saved 549 reports, and 40 of them were marked severe. Each one has an impact score, so a hard hit at low speed, like 9.8 at 13 km/h, stands out from a light one at 50 km/h.

These are reports of jolts, not confirmed potholes. A speed bump or a rough patch makes a jolt too, so the number is not a count of potholes. One drive can't tell the two apart. The test that will is driving the same road again and seeing which reports show up in the same places.

## Battery

The app keeps its own battery log. In my first test:

- **Recording a trip** used 7.3% of the battery per hour, measured over 57 minutes.
- **Waiting for a trip to begin** used 11.7% per hour, measured over about an hour.

Waiting cost more than recording. Watching for the start of a trip is what makes the app hands-free, so that's where I should look to save power. These are short tests on one phone, so treat them as a first look.

## What isn't done yet

- **Nothing is uploaded.** The reports stay on my phone, and I export them by hand.
- **The public map isn't live.** I've built a viewer, but there's no server joining reports from many phones.
- **The "several phones agree" step hasn't been tested**, because I'm the only person using it.

## Next

Drive the same roads again and see which reports repeat. Then let a few other people use it, so reports from different phones can confirm each other.
