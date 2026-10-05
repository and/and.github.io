---
layout: post
title: "Roasting Peanuts in the Oven (and Rescuing Soggy Ones)"
date: 2026-10-05
categories:
  - cooking
permalink: /cooking/roasting-peanuts/
excerpt: Roast raw peanuts at 175°C for 11 minutes, with a built-in timer. Plus how to cool them, and how to re-crisp soggy ones.
---

Roasting peanuts at home takes one baking sheet and 11 minutes in the oven. Here's the method, the reasoning behind each number, and what to do if you forget them on the counter and they go soft.

![Roasted peanut kernels spread in a single layer on a baking sheet]({{ "/assets/images/peanuts/hero-baking-sheet.svg" | relative_url }})

## In a hurry? Quick summary

| | Roast raw peanuts | Re-crisp soggy peanuts |
|---|---|---|
| **Quantity** | About 450 g (1 lb, roughly 3 cups), single layer | Same, single layer |
| **Temperature** | 175°C (350°F) | 150°C (300°F) |
| **Duration** | 11 min, no stirring needed in a true single layer | About 10 min, plus 5 more if still soft |
| **Cool before sealing** | 30-45 min | 30-45 min |

Done when golden brown and fragrant. They crisp up as they cool, so don't judge crunch while they're hot.

## Start a timer

Preheat to 175°C, spread the peanuts in a single layer, then start the timer.

<style>
.pt-wrap{display:flex;justify-content:center;margin:1.5rem 0}
.pt-card{width:100%;max-width:340px;border:1px solid rgba(127,127,127,.35);border-radius:16px;padding:1rem;background:rgba(127,127,127,.06);text-align:center}
.pt-card img{width:100%;max-width:260px;height:auto;border-radius:12px;display:block;margin:0 auto .5rem}
.pt-card h3{margin:.25rem 0 0;font-size:1.15rem}
.pt-spec{margin:.15rem 0 .5rem;font-size:.9rem;opacity:.8}
.pt-ring{position:relative;width:150px;height:150px;margin:.25rem auto}
.pt-ring svg{width:100%;height:100%;transform:rotate(-90deg)}
.pt-ring .bg{fill:none;stroke:rgba(127,127,127,.25);stroke-width:9}
.pt-ring .fg{fill:none;stroke:#c98a3b;stroke-width:9;stroke-linecap:round}
.pt-card.pt-done .pt-ring .fg{stroke:#3a8f4f}
.pt-time{position:absolute;inset:0;display:flex;align-items:center;justify-content:center;font-size:2rem;font-weight:600;font-variant-numeric:tabular-nums}
.pt-cue{min-height:3.6em;margin:.5rem 0;font-size:.95rem}
.pt-btns{display:flex;gap:.5rem;justify-content:center;flex-wrap:wrap}
.pt-btns button{font:inherit;font-size:.95rem;padding:.45rem .9rem;border-radius:999px;border:1px solid rgba(127,127,127,.5);background:transparent;color:inherit;cursor:pointer}
.pt-btns button.pt-go{background:#c98a3b;border-color:#c98a3b;color:#fff;font-weight:600}
.pt-btns button:focus-visible{outline:2px solid #c98a3b;outline-offset:2px}
</style>
<div class="pt-wrap">
<article class="pt-card" id="pt-timer" data-minutes="11" data-warn="120" data-add="1">
<img src="{{ "/assets/images/peanuts/raw-peanuts.svg" | relative_url }}" alt="Loose raw peanut kernels">
<h3>Raw peanuts</h3>
<p class="pt-spec">175°C (350°F) · 11 min · single layer, no stirring</p>
<div class="pt-ring"><svg viewBox="0 0 120 120" aria-hidden="true"><circle class="bg" cx="60" cy="60" r="52"></circle><circle class="fg" cx="60" cy="60" r="52"></circle></svg><div class="pt-time" role="timer" aria-label="Roasting timer">11:00</div></div>
<p class="pt-cue" aria-live="polite">Oven at 175°C and peanuts in a single layer? Press Start.</p>
<div class="pt-btns"><button type="button" class="pt-go">Start</button><button type="button" class="pt-reset">Reset</button><button type="button" class="pt-add">+1 min</button></div>
</article>
</div>
<noscript><p>The timer needs JavaScript. Roast raw peanuts for 11 minutes at 175°C (350°F).</p></noscript>
<script>
(function () {
  var AC = window.AudioContext || window.webkitAudioContext, actx = null;
  function wake() { try { if (AC) { actx = actx || new AC(); if (actx.state === "suspended") { actx.resume(); } } } catch (e) {} }
  function beep() {
    try {
      if (actx) { [0, 0.35, 0.7].forEach(function (t) {
        var o = actx.createOscillator(), g = actx.createGain(), n = actx.currentTime + t;
        o.type = "sine"; o.frequency.value = 880;
        g.gain.setValueAtTime(0.0001, n); g.gain.exponentialRampToValueAtTime(0.25, n + 0.02); g.gain.exponentialRampToValueAtTime(0.0001, n + 0.3);
        o.connect(g); g.connect(actx.destination); o.start(n); o.stop(n + 0.32);
      }); }
      if (navigator.vibrate) { navigator.vibrate([200, 100, 200]); }
    } catch (e) {}
  }
  var cues = {
    idle: "Oven at 175°C and peanuts in a single layer? Press Start.",
    run: "Roasting. No stirring needed if they are in a single layer.",
    warn: "Start checking now: look for golden brown and a roasted smell.",
    done: "Time! Take them out. They firm up as they cool, so wait 30-45 minutes before sealing."
  };
  var C = 2 * Math.PI * 52;
  function fmt(s) { var m = Math.floor(s / 60), r = s % 60; return (m < 10 ? "0" : "") + m + ":" + (r < 10 ? "0" : "") + r; }
  var card = document.getElementById("pt-timer");
  var startMin = parseInt(card.getAttribute("data-minutes"), 10);
  var warn = parseInt(card.getAttribute("data-warn"), 10);
  var step = parseInt(card.getAttribute("data-add"), 10);
  var go = card.querySelector(".pt-go"), reset = card.querySelector(".pt-reset"), add = card.querySelector(".pt-add");
  var time = card.querySelector(".pt-time"), cue = card.querySelector(".pt-cue"), fg = card.querySelector(".fg");
  var total = startMin * 60, remaining = total, endAt = 0, timer = null, state = "idle";
  fg.style.strokeDasharray = C;
  function draw() {
    time.textContent = fmt(remaining);
    fg.style.strokeDashoffset = state === "done" ? 0 : C * (1 - remaining / total);
    card.classList.toggle("pt-done", state === "done");
  }
  function setCue(t) { if (cue.textContent !== t) { cue.textContent = t; } }
  function stop() { if (timer) { clearInterval(timer); timer = null; } }
  function tick() {
    remaining = Math.max(0, Math.round((endAt - Date.now()) / 1000));
    if (remaining === 0) { stop(); state = "done"; go.textContent = "Start"; setCue(cues.done); draw(); beep(); return; }
    setCue(remaining <= warn ? cues.warn : cues.run);
    draw();
  }
  function run() { endAt = Date.now() + remaining * 1000; state = "running"; go.textContent = "Pause"; stop(); timer = setInterval(tick, 250); tick(); }
  go.addEventListener("click", function () {
    wake();
    if (state === "running") { stop(); state = "paused"; go.textContent = "Resume"; setCue("Paused."); return; }
    if (state === "done") { remaining = total = startMin * 60; }
    run();
  });
  reset.addEventListener("click", function () {
    stop(); state = "idle"; total = remaining = startMin * 60; go.textContent = "Start"; setCue(cues.idle); draw();
  });
  add.addEventListener("click", function () {
    wake();
    if (state === "done") { total = remaining = step * 60; run(); return; }
    total += step * 60; remaining += step * 60;
    if (state === "running") { endAt += step * 60 * 1000; tick(); } else { draw(); }
  });
  draw();
})();
</script>

## The basic method

1. Spread raw peanuts in a single layer on a baking sheet.
2. Roast at **175°C (350°F)** for about **11 minutes**. If every peanut is in a true single layer, you don't need to stir; they brown evenly on their own. If they're piled in places, stir once or twice.
3. They're done when golden brown and fragrant. They'll firm up more as they cool.
4. If you want them salted, toss with salt while they're still warm.
5. Cool completely before eating or storing.

11 minutes is what works in my oven with a single layer and no stirring. Ovens vary, and a crowded pan will take longer (15-20 minutes with stirring).

**Tip:** Start checking around the 9-minute mark. Peanuts go from perfect to burnt quickly. Let a test batch cool a couple of minutes before tasting, because hot peanuts taste underdone even when they're not.

![Raw peanut kernels on the left, golden roasted kernels on the right]({{ "/assets/images/peanuts/raw-vs-roasted.svg" | relative_url }})
*Aim for the golden-brown color on the right.*

## Why 175°C?

It's the sweet spot. It's hot enough to drive off moisture and kick off the Maillard reaction, the browning chemistry that creates that roasted flavor, at a decent pace. But it's not so hot that the outside scorches before the middle cooks through.

Peanuts are dense and oily. Go much higher and the outside burns while the center still tastes raw. Go lower and you wait a long time and end up with a "dried" texture rather than a true roast.

## Cool them before sealing

Let the peanuts sit at room temperature for **30-45 minutes** before putting them in a lidded container.

They keep cooking slightly from residual heat, and they release moisture as they cool. Seal them while warm and that trapped moisture makes them soggy and stale-tasting much sooner.

**Quick check:** they're ready to seal once the container feels like room temperature to the touch, not just "not hot."

![Roasted peanuts in a jar with a closed lid]({{ "/assets/images/peanuts/sealed-jar.svg" | relative_url }})
*Seal only once the container feels like room temperature.*

## If they've already gone soggy

Forgot them on the counter and they've gone soft? They're easy to recover:

1. Spread them back on a baking sheet in a single layer.
2. Bake at **150°C (300°F)** for about **10 minutes**.
3. Cool fully again (30-45 minutes) before sealing.
4. If they still feel soft once cool, give them another 5 minutes.

Peanuts crisp up as they cool, so don't judge crunch while they're warm.

### Why 150°C and 10 minutes?

The goal this time is different. The peanuts are already cooked, so you're not roasting them, only evaporating the moisture they absorbed. A lower temperature does that without pushing them toward bitter, over-roasted territory (their sugars are already partly caramelized from the first round).

Ten minutes is enough for gentle moisture removal without drying them out or scorching the surface.

**Rule of thumb:** first roast = higher heat, longer time, cooking through. Re-crisping = lower heat, shorter time, just removing surface moisture.
