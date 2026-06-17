---
layout: post
title: Welcome back - let's build something?
byline: Resurrecting this blog to document a learning journey on Agentic systems design
featured: true
banner: /images/2026-06-11-welcome-back-banner.svg
series: Building an Agentic Golf Caddie
---

Next post: Context Window Management and the impact on Agentic design.

The instinct: a round is naturally one continuous thing — 18 holes, one conversation, feels obviously right
The arithmetic that breaks it: 70–100 shots per round, each a turn (sometimes with tool calls), compounding resend cost across a growing messages slice — back-of-envelope token cost by hole 15
The overcorrection: stateless per shot fixes the bloat but loses in-hole continuity, and quietly reintroduces cost elsewhere — a tool call to re-establish "what happened on this hole" on every single shot
The actual unit of play: the hole, not the round — bounded conversation (2–6 exchanges typical, a triple-bogey par 5 as the worst case), fetch hole context once, reuse it across shots, discard and start fresh at the next tee
The reframe: this is really a point about scoping agent state to the natural unit of the domain, not a golf-specific trick — applicable well beyond this project
Honest coda: you haven't built Stage 3 yet, so this is a design decision made on paper before code — worth flagging as "here's my reasoning, we'll see if it survives contact with a real round"

