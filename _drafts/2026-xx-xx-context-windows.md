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

Talk about how the context window grows over time through repeated calls, tool loops, etc.
First run with standard - 40k input token, 1700 output tokens 24s
BASIC caching: Turning on ephemeral caching - 14.2K, 1681 output, 26.6k Cache writes
Still using ~3k input tokens per call
Context block - static v dynamic content. Not being cached.
Solution is to split the context block into static v dyanmic content

Automatic Caching v Explicit Breakpoints
but if we only write to the cache and dont read, its just more expensive 1.25x on writes 

Cache Controls on System and Context Prompts
    Next version: Remvoe dynamic content out of the context block (holes played, score behind.)
    * System prompt — TextBlockParam.CacheControl now set. The system prompt is identical on every call, so Anthropic will serve it from cache after the first request (5-minute TTL). You'll see cache_read_input_tokens go up on subsequent calls.
    * Context block — pulled out of the concatenated string into its own ContentBlockParamUnion{OfText: &TextBlockParam{...}} with its own CacheControl. This creates a cache breakpoint after the context but before the user question. The cache key covers everything up to that breakpoint: system + context. Within a single Run call, as the tool-use loop adds more messages, the context block never changes position so it stays cache-hot.
Didnt really work

- Every Call 1: input=2837, cache_create=0, cache_read=0 — no caching at all
- Every Call 2 (after prefill {): input=0, cache_create=5600+, cache_read=0 — everything cached, but there's no Call 3 to read it

What a tool-level breakpoint does

If you mark the last tool with CacheControl, the cache breakpoint sits at the end of the tools block:

[ system ] [ tool1 ] [ tool2 ← breakpoint ] [ messages... ]

The cache entry covers system + tools. That's ~2200 tokens of content that is identical across every single shot recommendation — same system prompt, same two tool definitions, every time.

---
The expected flow if it works

Shot 1, Call 1 — first ever call, no cache exists yet:
system + tools + userMsg(H1S1)
     ↑ breakpoint
cache_create = ~2200 tokens (system+tools written to cache)
input = ~637 tokens (userMsg, not covered by breakpoint)

Shot 1, Call 2 — after tool results, ends with assistant prefill:
system + tools + userMsg + assistant[tool_use] + user[results] + assistant["{"]
     ↑ breakpoint
cache_read = ~2200 (system+tools already in cache)
cache_create = ~3500 (the new longer suffix also gets cached via auto-cache)

Shot 2, Call 1 — different hole, different userMsg, but same system+tools:
system + tools + userMsg(H1S2)
     ↑ same breakpoint, same content
cache_read = ~2200 ← THIS is what we want
input = ~637 (only the new userMsg)

Every subsequent shot reads ~2200 tokens from cache instead of paying full price for them.

GetHoleStats Tool Call:
    serialization of tool calls: {"Int64":4,"Valid":true} - 3x more tokens than just 4



Anthropic prompt caching works by prefix matching. A cache hit only occurs when the start of a new request exactly matches a previously cached prefix. Auto-cache only triggers when a request ends with an assistant message — which is why you see cache writes on Call 2 (ends with the { prefill) but never on Call 1 (ends with a user message).

Every Call 1 looks like:
system + tools + contextBlock + per-hole question

The per-hole question changes every shot, so the prefix is always unique. Even though system+tools+contextBlock is identical across all 18 holes, there's never a cache entry for it because:
- Explicit markers don't trigger cache writes on user-ending requests (we've proven this empirically)
- Auto-cache only fires on assistant-ending requests

The fix: Pre-warm the contextBlock prefix once, ending with an assistant message, so it exists in cache. Then every hole call starts with that same prefix and gets a cache hit for it.

Structurally, every hole call would look like:
system + tools
  User: contextBlock          ← cached from pre-warm
  Assistant: "Ready."         ← the cache anchor
  User: per-hole question     ← only these ~50 tokens are new input

This is actually how Anthropic intends caching to work — through multi-turn conversation continuation. We're just doing it synthetically with a warm-up call.

balancing cost benefits of caching in higher order models v running in lower models.

## Ending

Same priniciples apply for other agentic applications - need to consider what to send over the wire, how to cache it.
Context context context