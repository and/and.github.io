---
layout: post
title: "AI Adoption at Rocketium"
date: 2026-10-26 14:30:00 +0530
permalink: /ai/
sitemap: false
---

<style>
.ath-ba{width:100%}
.ath-ba th[scope=col]{background:var(--subtle)}
.ath-ba th[scope=row]{font-weight:600;width:24%}
.ath-ba td.ath-after{color:var(--text)}
.ath-ba td.ath-before{color:var(--muted)}
@media (max-width:600px){
  .ath-ba thead{position:absolute;width:1px;height:1px;overflow:hidden;clip:rect(0 0 0 0)}
  .ath-ba,.ath-ba tbody,.ath-ba tr,.ath-ba th,.ath-ba td{display:block;width:auto}
  .post-content .ath-ba tr{border:1px solid var(--rule);border-radius:10px;margin-bottom:12px;overflow:hidden}
  .post-content .ath-ba th[scope=row]{width:auto;background:var(--subtle);border:0;border-bottom:1px solid var(--rule)}
  .post-content .ath-ba td{border:0;padding:8px 12px}
  .post-content .ath-ba td::before{content:attr(data-label);display:block;font-size:.72rem;font-weight:600;letter-spacing:.06em;text-transform:uppercase;color:var(--faint)}
  .post-content .ath-ba td.ath-before{border-bottom:1px dashed var(--rule)}
}
.ath-tl{margin:8px 0 28px}
.post-content .ath-tl ol{list-style:none;margin:0;padding:0;position:relative}
.ath-tl ol::before{content:"";position:absolute;top:12px;bottom:12px;left:calc(var(--ath-w) + 33px);width:2px;background:linear-gradient(var(--rule),var(--accent))}
.ath-tl{--ath-w:7.5rem}
.post-content .ath-tl li{margin:0;position:relative}
.ath-year{display:grid;grid-template-columns:var(--ath-w) 40px 1fr;column-gap:14px;padding:14px 0 6px}
.ath-year span{grid-column:2;justify-self:center;font-weight:700;font-size:.85rem;letter-spacing:.04em;color:var(--surface);background:var(--accent);border-radius:999px;padding:2px 10px}
.ath-ev{display:grid;grid-template-columns:var(--ath-w) 40px 1fr;column-gap:14px;align-items:start;padding:6px 0}
.ath-when{font-family:var(--mono);font-size:.78rem;color:var(--faint);text-align:right;padding-top:10px}
.ath-dot{width:40px;height:40px;border-radius:50%;background:var(--surface);border:2px solid var(--rule);display:grid;place-items:center;color:var(--accent);position:relative;z-index:1}
.ath-dot svg{width:20px;height:20px;fill:none;stroke:currentColor;stroke-width:1.8;stroke-linecap:round;stroke-linejoin:round}
.post-content .ath-ev p{margin:0;padding:8px 14px;background:var(--surface);border:1px solid var(--rule);border-radius:10px;line-height:1.5}
.ath-now .ath-dot{background:var(--accent);border-color:var(--accent);color:var(--surface)}
.post-content .ath-now p{border-color:var(--accent)}
@media (max-width:600px){
  .ath-tl ol::before{left:19px}
  .ath-year,.ath-ev{grid-template-columns:40px 1fr;column-gap:12px}
  .ath-year span{grid-column:1 / 3;justify-self:start}
  .ath-when{grid-column:2;grid-row:1;text-align:left;padding:0 0 4px}
  .ath-dot{grid-column:1;grid-row:1 / 3}
  .ath-ev p{grid-column:2}
}
</style>

For the past two and a half years I've been inside the kind of shift AI enablement roles exist to drive. It happened at Rocketium, a company of 50+ people, not a large organization.

## Before and after

<table class="ath-ba">
<thead><tr><th scope="col"></th><th scope="col">Before</th><th scope="col">After</th></tr></thead>
<tbody>
<tr><th scope="row">Non-tech team ideas</th><td class="ath-before" data-label="Before">Pitched on a PPT or wireframe</td><td class="ath-after" data-label="After">Working project by day 3 of the hackathon</td></tr>
<tr><th scope="row">LLMs in engineering</th><td class="ath-before" data-label="Before">Nice-to-have for code snippets</td><td class="ath-after" data-label="After">Helping write full features</td></tr>
<tr><th scope="row">Image analysis build</th><td class="ath-before" data-label="Before">3+ months, 1,000+ training images</td><td class="ath-after" data-label="After">Over a weekend</td></tr>
<tr><th scope="row">Taking a tool company-wide</th><td class="ath-before" data-label="Before">Engineering set up hosting and cloud</td><td class="ath-after" data-label="After">Non-developers ship end to end with Lovable</td></tr>
<tr><th scope="row">Internal tools</th><td class="ath-before" data-label="Before">Bought SaaS, like a ~$1k/month task management tool</td><td class="ath-after" data-label="After">Teams build their own, and several SaaS tools were dropped</td></tr>
<tr><th scope="row">Security checks</th><td class="ath-before" data-label="Before">I checked each tool by hand</td><td class="ath-after" data-label="After">Recommended tools, automated code scanning, code review, device protection</td></tr>
</tbody>
</table>

## Timeline

<figure class="ath-tl" aria-label="Timeline of AI adoption at Rocketium, 2023 to 2026">
<ol>
<li class="ath-year" aria-hidden="true"><span>2023</span></li>
<li class="ath-ev"><span class="ath-when"></span><span class="ath-dot"><svg viewBox="0 0 24 24" aria-hidden="true"><rect x="3" y="4" width="18" height="12" rx="1.5"/><path d="M12 16v4M8 20h8"/></svg></span><p>Non-tech teams pitch ideas on PPTs and wireframes.</p></li>
<li class="ath-ev"><span class="ath-when">Late 2023</span><span class="ath-dot"><svg viewBox="0 0 24 24" aria-hidden="true"><path d="M12 3l1.8 5.2L19 10l-5.2 1.8L12 17l-1.8-5.2L5 10l5.2-1.8z"/><path d="M19 15v4M17 17h4"/></svg></span><p>The engineering team starts adopting AI more, and finds it empowering.</p></li>
<li class="ath-year" aria-hidden="true"><span>2024</span></li>
<li class="ath-ev"><span class="ath-when"></span><span class="ath-dot"><svg viewBox="0 0 24 24" aria-hidden="true"><path d="M13 2 4 14h7l-1 8 9-12h-7z"/></svg></span><p>Yearly hackathon. People Ops and Product build working tools, and almost everyone has a project by day 3.</p></li>
<li class="ath-ev"><span class="ath-when">Summer 2024</span><span class="ath-dot"><svg viewBox="0 0 24 24" aria-hidden="true"><rect x="6" y="6" width="12" height="12" rx="2"/><path d="M9 2v4M15 2v4M9 18v4M15 18v4M2 9h4M2 15h4M18 9h4M18 15h4"/></svg></span><p>Claude 3.5 Sonnet and GPT-4o. We start realising LLMs can write good code.</p></li>
<li class="ath-ev"><span class="ath-when">Late 2024</span><span class="ath-dot"><svg viewBox="0 0 24 24" aria-hidden="true"><path d="M7 18h10a4 4 0 0 0 .5-7.97A6 6 0 0 0 6.1 9.5 4.25 4.25 0 0 0 7 18z"/></svg></span><p>Teams ask engineering to host their tools. Rocketium adopts Lovable internally.</p></li>
<li class="ath-year" aria-hidden="true"><span>2025</span></li>
<li class="ath-ev"><span class="ath-when"></span><span class="ath-dot"><svg viewBox="0 0 24 24" aria-hidden="true"><circle cx="12" cy="12" r="9"/><path d="m8 12 3 3 5-6"/></svg></span><p>Yearly hackathon. Everyone ships a working tool end to end. The image build takes a weekend.</p></li>
<li class="ath-ev"><span class="ath-when">Mid-2025</span><span class="ath-dot"><svg viewBox="0 0 24 24" aria-hidden="true"><circle cx="6" cy="6" r="3"/><circle cx="6" cy="18" r="3"/><path d="M8.1 8.1 20 20M8.1 15.9 20 4"/></svg></span><p>Teams replace SaaS with their own tools. The task management tool is dropped.</p></li>
<li class="ath-year" aria-hidden="true"><span>2026</span></li>
<li class="ath-ev"><span class="ath-when">January 2026</span><span class="ath-dot"><svg viewBox="0 0 24 24" aria-hidden="true"><path d="M9 3h6M10 3v6L4.5 19a1.5 1.5 0 0 0 1.3 2h12.4a1.5 1.5 0 0 0 1.3-2L14 9V3"/><path d="M7 15h10"/></svg></span><p>Co-founders start Labs.</p></li>
<li class="ath-ev"><span class="ath-when">April 2026</span><span class="ath-dot"><svg viewBox="0 0 24 24" aria-hidden="true"><path d="M12 3 4 6v6c0 4.5 3.4 8 8 9 4.6-1 8-4.5 8-9V6z"/><path d="m9 12 2 2 4-4"/></svg></span><p>Security structure in place.</p></li>
<li class="ath-ev ath-now"><span class="ath-when"></span><span class="ath-dot"><svg viewBox="0 0 24 24" aria-hidden="true"><path d="M3 11 12 4l9 7"/><path d="M5 10v10h14V10"/><path d="M10 20v-6h4v6"/></svg></span><p>Most internal tools that depended on external tools are replaced by in-house tools built by each team.</p></li>
</ol>
</figure>

## How it started

I was the Engineering Manager responsible for cloud infra, compliance (ISO, SOC 2) and quality. Around late 2023, the engineering team started adopting AI more, and we found it empowering. So at our 2024 hackathon, we brought in the People Ops and Product teams to build working tools too. There was fear and reluctance, but by day 3 almost everyone had a working project. A year earlier, non-tech teams would have presented an idea on a PPT or a wireframe.

That summer, with Claude 3.5 Sonnet and GPT-4o, we started realising LLMs could write good code. They went from a nice-to-have for generating snippets to helping write full features.

## Teams building their own tools

After that, more teams came to engineering for help. They had tools that worked on their own computers and wanted them available to the whole company. That mostly meant hosting and cloud setup, which was my area. Then we adopted Lovable internally, and non-developers could build end to end without thinking about hosting. At our 2025 hackathon, everyone had a working tool end to end. Image analysis had improved too. A year earlier, one build had taken us 3+ months and 1,000+ training images. Now we could build the same thing over a weekend.

Over the following months, teams built their own tools instead of buying SaaS. People Ops and the business teams dropped several SaaS tools. We even dropped our company task management tool, which cost almost $1k a month.

I was one of the managers those teams reached out to. Over time I guided them to build their own tools, some for personal problems and some for their teams. It was a joy to see people feel empowered to solve their own challenges.

## Labs

Engineering was shipping more than the roadmap planned, so our co-founders started a new team called Labs to experiment and take new bets. Labs had a business analyst, a strategy consultant, two engineers, me and the co-founders. We built tools for different teams and tested which areas of the creative ops space we could grow into. A lot of that happened in late-night hackathons and daily connects, bouncing ideas off my co-founder and helping him realise his ideas.

## Security and compliance

Through all of it I played several roles at once: manager, infra owner, the person accountable for ISO and SOC 2, and coach to peers on non-tech teams. One of the biggest challenges was bringing a sense of security and privacy to the non-tech teams. They were elated to see their ideas come to life, but every tool still had to be checked for security and privacy. At first I did those checks myself. Over time we put a structure in place: recommended tools, automated code scanning, code review and device protection tools.

## Looking back

Looking back, here's what I'd do differently:

- Build centralised data sources that every team can build tools on.
- Choose a few vetted tools for teams to build on from the start, to avoid compliance issues.
- Set up automated tests and checks from the start, so code from anyone who builds gets checked automatically.
- Hold build sessions and hackathons more often.
- Hold office hours where people can drop in for help.

## What I bring

In an AI enablement role, I'd be a guide, but I'd also get my hands dirty, build alongside each team, and slowly remove the need for me. I come in, help everyone adopt AI, and move on. I believe people are talented and creative, and with more capable tools, everyone can amplify their talent and creativity.

I'd welcome a conversation about what that could look like at your company.

ānand  
[linkedin.com/in/anandhms](https://www.linkedin.com/in/anandhms/)
