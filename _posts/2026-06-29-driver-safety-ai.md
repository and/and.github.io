---
layout: post
title: "A dashcam that runs entirely in the browser"
date: 2026-06-29 21:49:02 +0530
categories: [utilities, cars]
permalink: /utilities/driver-safety-ai/
excerpt: "A browser-based driver-assistance prototype that turns a phone camera into a dashcam with collision, lane-departure and drowsiness alerts, all on the phone. Testing on Indian roads showed where it falls short."
---

I wanted to see how much of a driver-assistance system a phone could run on its own. The result is ADAS Road Monitor, a prototype that turns a phone camera into a dashcam with collision, lane-departure and drowsiness alerts. It runs in the browser. There is no server and no account, and nothing leaves the phone.

It is a prototype, not a safety device. Please don't rely on it to keep you safe.

**[Try it here](https://and.github.io/dashcam/)**. Open it in Chrome or Firefox, allow the camera and location, tap **START ADAS**, and mount the phone so the camera faces the road.

## What it does

- **Collision warning.** An on-device model (COCO-SSD, running in TensorFlow.js) spots people, cyclists, motorcycles, animals, vehicles, traffic lights and stop signs. Each one gets a threat level from 0 to 100, based on its size, its position and how fast it is closing in. The screen border changes colour (green, amber, orange, red) and a beep sounds as the level rises.
- **Lane departure.** If you switch it on, it looks for lane markings and announces by voice when the car drifts across one.
- **Drowsiness check.** About every 45 seconds it briefly turns on the front camera and checks whether your eyes are open. If they stay shut, it flashes a warning, sounds an alarm and tells you to pull over safely.
- **Speed.** It shows speed from GPS, and estimates it from the camera's motion when GPS is weak.
- **Incident clips.** It keeps a rolling video buffer, 10 minutes by default, and saves the clip when it detects a serious event. Saved clips are in a gallery, and past trips show on a map with their events.

## What testing on Indian roads showed

I took it out on real roads. Two things went wrong.

**The phone overheated.** Running detection, motion estimation and lane detection constantly was too much for a phone. In my test it overheated and shut down in under 10 minutes. An early fix capped detection at about six or seven frames per second instead of running on every screen refresh. That cuts the load, but a heavy model is still heavy.

**False alerts made me stop trusting it.** My car's bonnet is visible at the bottom of the camera frame, and it kept triggering proximity alerts. I added two sliders to move the caution and danger lines, and the app now ignores anything that sits below the danger line. More broadly, a system that cries wolf trains its driver to ignore it. That is a worse problem than missing the occasional alert, because it takes away the trust the whole thing depends on.

## What I'd do next

- Try lighter models that put less heat into the phone.
- Filter alerts more carefully, so that the ones that do fire are the ones worth looking at.

## Using it

The app needs a modern browser with WebGL, a camera, and a secure (HTTPS) page. Location is optional but improves the speed reading. Settings let you change the sensitivity, switch lane detection and the driver check on or off, move the detection lines, and choose how long the recording buffer is.

The code is on [GitHub](https://github.com/and/dashcam).
