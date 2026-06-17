---
layout: post
title: Harness Engineering
byline: Resurrecting this blog to document a learning journey on Agentic systems design
featured: true
banner: /images/2026-06-11-welcome-back-banner.svg
series: Building an Agentic Golf Caddie
---

This is also a nice, more technical companion angle for the same blog post — "scoping state to the hole" is the design narrative, and "here's what that actually meant for a stateless Go API talking to a stateless SDK" is the honest implementation reality that backs it up, including the bit where you realize the SDK was never going to do this for you.

The seduction of the messages slice: it's right there, the SDK practically begs you to just serialize it — []anthropic.MessageParam, json.Marshal, done
Why that's the wrong default for this domain: the model's own prior reasoning isn't actually what the next shot needs — it needs facts, not the conversational artifact of a previous take on those facts
The reframe: state persistence isn't "save the conversation," it's "what's the minimal factual record that lets me reconstruct an equivalent next turn" — and that record looks like your existing schema (rows), not like conversation logs
Where verbatim does still earn its place: the short-lived, single-decision back-and-forth ("what if I laid up instead") — scoped tightly enough that it never needs to outlive the moment
The general lesson: frameworks hide this decision from you; building on the raw SDK forces you to make it explicitly — which is exactly the "explicit over magic" instinct that's been steering every other decision in this project

Here's the harness-engineering framing, in Sean's voice:
What "harness engineering" actually means
The Anthropic SDK gives you one function that does one thing: take a full history, return the next turn. No session, no memory, no concept of "this conversation" persisting anywhere on Anthropic's side. Everything that makes an agent feel like it remembers anything — across a shot, a hole, a round — is something I have to build. That's the harness: the layer between "the model can reason" and "the system behaves like it has continuity."
It's easy to skip past this if you're using a framework, because the framework's already made the decision for you and called it "memory" or "sessions" — and you never have to look at what's underneath. Building directly on the SDK means I don't get that luxury. I have to decide, explicitly, what state exists, where it lives, and what form it takes. That's not overhead. That's the actual design problem.
Why it mattered here specifically
The naive version — one long conversation for the whole round — looks obviously right until you do the arithmetic. 70 to 100 shots, each one potentially a multi-turn tool-use exchange, all of it resent on every single call because that's how the API works. By the back nine I'm paying to resend a transcript nobody's reading.
The opposite instinct — wipe the slate clean every shot — fixes that, but quietly creates a different cost. Every shot now needs a fresh tool call just to re-establish what's already happened on this hole, because there's no conversation left to remember it.
Neither of those is a "harness" decision so much as an absence of one — just defaulting to the two extremes the API naturally falls into. Scoping the conversation to a hole was the actual harness decision: a deliberate, bounded unit that matches how the game is actually played, not how the API happens to be shaped.
The options, and what each one actually costs
Once I'd decided what unit of state I wanted, the next question was how to persist it between stateless HTTP calls, since my Go API doesn't hold anything in memory between requests either.

Persist the messages slice verbatim. Serialize the conversation, store it, reload it next shot. Highest fidelity — the model sees its own prior reasoning exactly as it happened. But it's also a second, parallel notion of "history" that lives outside my schema, isn't queryable, and carries information the next shot doesn't actually need.
Reconstruct from structured facts. Store what happened — club, distance, lie, outcome — as rows, the same as everything else in this system. Rebuild a fresh prompt from those facts each time. Lower fidelity to the model's own reasoning trail, but it's smaller, debuggable, and consistent with how I'm storing everything else.
Hybrid. Keep the raw conversation for the life of the hole, discard it once the hole's holed out, and only the structured summary survives into the permanent record.

Where I landed, and why
The structured-facts version won, and it won for the same reason most of the decisions in this build have gone the same direction: it keeps the state in the same shape as everything else in the system — rows in SQLite I can inspect and reason about independently of whatever conversation happened to produce them. The verbatim approach works, but it's magic by another name — a black box of conversation state sitting outside the schema I've built everything else around.
What the next shot actually needs isn't the model's previous paragraph about wind and dispersion. It's the fact that the last shot was a driver, it went left, and there's 140 yards left. Facts, not the conversational artifact of a previous take on those facts.
There's one place verbatim conversation still earns its keep: the short, single-decision back-and-forth, where the player pushes back on a recommendation before committing to a shot. That's tightly scoped enough that it never needs to outlive the moment, and it's a genuinely different kind of state than the hole-spanning record.
The general lesson
None of this is golf-specific. Any agent built directly on a raw model API hits the same fork: the API hands you a stateless function, and the state model is entirely yours to design. Frameworks hide that fork from you. Building without one means I have to actually stand at it and choose — which is exactly the trade I made going in, and exactly why it's worth writing about honestly rather than presenting the final architecture like it was obvious from the start.