---
layout: post
title: The agent didn't lie, it just didnt know
byline: Garbage in, confidence out
featured: false
banner: /images/2026-06-11-welcome-back-banner.svg
series: Building an Agentic Golf Caddie
---

POST: [Working title] "The Agent Didn't Lie. It Just Didn't Know."

(or something in that territory — the point is: this isn't a hallucination problem)

Opening — no warmup

The 5 iron story. Drop straight into it. You asked for a recommendation. It said 5 iron, avoid the bunker left. The reasoning was coherent. You almost trusted it. Then you realised driver is the obvious play — the bunker is nowhere near where driver lands. The agent wasn't reasoning badly. Its map of the hole just stopped at the hazard.
One short declarative to land it: "That's a different kind of wrong."

Section: Hallucinations get the press. This is worse.

Why confident-and-wrong-but-plausible is the dangerous failure mode. A hallucination has tells — you can fact-check it. This you can only catch if you already know the answer, which defeats the point. The agent's epistemic posture doesn't change based on completeness of data. It reasons the same way whether it has 20% of the context or 100%.
Frame the general principle: agents don't know what they don't know, and they don't tell you when they're reasoning from an incomplete picture.

Section: The agent's world is what you built

Transition into the data model — but framed as the boundary of the agent's cognition, not as infrastructure. What it can reason about is exactly what you've captured. Nothing more.
Concrete data points to include:

What was actually in the data to start (scorecard-level: score, GIR, fairways, putts)
What that lets the agent reason about (scoring patterns, stat tendencies)
What it can't reason about without more (hole geometry, hazard placement, safe landing zones)
The shot reconstruction work — 31 rounds, ~2,500 shots inferred from scorecards — and what that buys you vs. what it can't give you

Short aside on the source field — the data knows how confident it should be, even if the agent doesn't always surface that.

Section: How much is enough? The classification problem

This is the uncomfortable design question. You can't capture everything. Where do you stop?
Structure it around the actual tradeoffs:

Too little: plausible but wrong (the 5 iron)
Too much: you're writing a yardage book in a database — different project, different lifetime
The line is arbitrary, but drawing it explicitly is better than not drawing it

What you actually did: hand-classified POIs from the St Michaels yardage book. Plain language labels, structured tagging, no polygon geometry. Concrete enough to be useful, scoped enough to be achievable.
The honest admission: this covers the hazards you thought to include. The agent still doesn't know things you didn't classify. You've narrowed the failure surface, not eliminated it.

Section: What this means for the recommendations

Brief. What the agent can now do vs. what remains out of scope. The pre-round game plan it can produce is genuinely useful — but bounded. You know the bounds. The player knows the bounds. That transparency is a design choice, not a consolation prize.

Ending — short, lands on the argument

Something like: the data layer isn't the boring part of an agentic system. It's the part that determines what the agent is actually allowed to know. Get that wrong and you don't get errors — you get confident advice from a caddie who's never seen the back nine.

A few notes on what's not in here:

No schema diagrams or SQL in this post — that's reference material, not narrative
The reconstruction methodology detail probably lives in a companion post or appendix
The green feature / segment vocabulary work could get a mention but shouldn't take over — it's an example of the classification decision, not a separate topic