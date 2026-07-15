---
layout: post
title: Context Design - How Much is Too Much?
byline: The learning that How you provide the information is sometimes just as important as What you provide
featured: true
banner: /images/2026-06-11-welcome-back-banner.svg
series: Building an Agentic Golf Caddie
mermaid: true
---

One of the first decisions that I needed to make when starting out with the Agentic Golf Caddy was how to actually interact the AI model.

I wanted the Agentic Caddy to be callable for a shot at any point on the course - describe the situation, leverage the data we have on the player, the course and the hole and provide a recommendaiton based on the situation, historical performance and trends.

All of this data is critical context that the Caddy needs to make it's recommendation.

But how much information should be provided? Is it always needed or should the agent _decide_ when to get it?

Once we've landed on an approach there, how timely should this information be?  Should we persist a session for the whole round or just within the context of a single request?

These decisions are somewhat simple on their own when thinking in traditional application architectural patterns, however they pose a unique challenge for Agentic Systems when you need to deal with Context Windows.

## What is the Context Window

If you read the [first post]({% post_url 2026-06-24-agentic1-fundamentals %}) in this series, all the interactions that you have with an Large Language Model is just sending structured text over the wire.

In a sense, you can think of every message that you send to Claude or ChatGPT as the first time the LLM has recevied a message from you. 

It's a one shot approach - send a message and get a response.

If you need to provide more information or a history of the conversation - say for a follow up question, then you send all previous messages over the wire so the model can reason over inputs and outputs and generate you an appropriate response.

This works up until the moment model hits a limit on the number of tokens that can be processed in context and hits the limit of it's Context Window.

It's important to understand this once we start talking about the application itself, as the size of the context that is continually passed back and forth, and how quickly this can grow, directly relates to not only the cost of your API calls but also critically, the latency of the responses.

## The Application as an AI Harness

[Harness Engineering](https://www.anthropic.com/engineering/harness-design-long-running-apps) is another hot topic within the world of agentic system design.

You can think of a harness as any application (either web, thick or command-line-interface) which you use to interact with an agentic system - so think of your Claude Code, ChatGPT mac application or the Google Gemini functionality built into Search as different forms of harnesses providing differing capabilities.

But all harness generally provide a similar set of capabilities:
- Identity & Access Management - API access, permissions, etc
- Input Controls - Human and System interaction
- Memory Management - Initial load and management of the context window
- Session Management - Persistence 
- Tooling Capabilities - extensibility points
- Guardrails and Policy - content filters, budget limits, approvals, etc
- Observability - logging and monitoring, traceability
- Error Handling and Retry Logic

Now, not all harnesses provide all of these capabilities but when designing how the Agentic Golf Caddy was going to function and interact with the LLM, we needed to consider all of these in isolation and how they work together.

| Capability | Decision Point |
| --- | --- |
| Identity & Access Management | Not implemented yet...naughty I know. | 
| Input Controls | Access to the Agent channeled through a local go AgentService.  Human input provided through a simple, text based TUI for now.  | 
| Memory Management | Golfer records their rounds and shots into a local SqlLite database which the AgentService queries to build the context - providing the course, handicap, clubs available, distances, etc straight to the model. | 
| Session Management | The harness allows the golfer to play a full or partial round as well as resume a round that's in progress.  Each request for advice however is completely standalone with no long-running session maintained outside what is achieved through context management. | 
| Tooling Capabilities | Two tools are made available for the first version - get_hole_features to describe the current hole's layout and hazards and get_hole_stats which returns the players historical performance on a specific hole.  The tools are made available and the agent decides if the tool is needed which the harness is responsible for running and providing the results back to the model. | 
| Guardrails and Policy | None. | 
| Observability | Logging and tracing implemented throughout - both in terms of traditional application logging but also Anthropic SDK telememtry to capture usage metrics and telemetry data. | 
| Error Handling and Retry Logic | Nothing specific yet outside standard Golang error handling practices. | 

## Tooling and The Trap of Agency

Once we had a working harness where we could simulate playing a few holes and getting caddy recommendations, it became clear from the telemetry and log files that something was a little off.

Every request for advice was taking around 5 seconds and while the token usage didnt seem crazy at the time (~8000 input tokens per request), I was noticing that every call to the agent was making multiple round-trips and it was all to do with the tools.

The decision to expose these capabilities as tools was intentional - agentic systems should have _agency_ and be empowered to decide what information is needed to be pulled in - so in theory this makes sense.  However, where we run into problems is situations like this where the tools absolutely must be called for the agent to be able to do it's thing.

Then it's just inefficient - as you can see in the diagram below every call to the Anthropic API for a tool call sends not just the results of the tool execution, but also, all previous messages sent.  In an application of this size, it's not a huge problem but still highly inefficient.

```mermaid
sequenceDiagram
    participant CLI as CLI
    participant Svc as AgentService
    participant Strat as GetHoleStrategy
    participant AgentLoop as Agent.Run
    participant API as Anthropic API
    participant DB as DB

    CLI->>Svc: GetRecommendation(hole, distance)

    rect rgb(230, 242, 255)
        note over Svc: Building Context BLock
        Svc->>DB: ListCompletedRoundsByPlayer
        DB-->>Svc: round history
    end

    Svc->>Strat: GetHoleStrategy(req)

    rect rgb(230, 242, 255)
        note over Strat: Registering Tools
        Strat->>Strat: RegisterTool(get_hole_stats)
        Strat->>Strat: RegisterTool(get_hole_layout)
        Strat->>Strat: SetFinalTool(submit_recommendation)
    end

    rect rgb(230, 242, 255)
        note over Strat,DB: Running Agent Loop

        Strat->>AgentLoop: Run(ctx, contextBlock, prompt)

        loop until model calls submit_recommendation
            AgentLoop->>API: Messages.New(messages, tools)
            API-->>AgentLoop: ToolUse: get_hole_stats / get_hole_layout
            AgentLoop->>DB: GetHoleStats / ListPOIsByHoleAndTee
            DB-->>AgentLoop: rows
            AgentLoop->>AgentLoop: append tool_result, loop again
        end

        AgentLoop->>API: Messages.New(messages, tools)
        API-->>AgentLoop: ToolUse: submit_recommendation
        AgentLoop-->>Strat: AgentResponse{Response, Usage}
    end
    Strat-->>Svc: GetHoleStrategyResponse
    Svc-->>CLI: Strategy, Club, Reasoning
```

## Baseline the Performance

At this stage, this was really a hypothesis - to be honest I didnt really know how much this was affecting the system?

Was it really something that should be fixed? If we could fix it, what kind of performance improvements could be made? Are the changes worth it?

Since this is a learning project, the answer is absolutely yes - lets dig in.

To really test whats going on, I built a small Golang harness that simulated a golfer playing a round and asking for advice on each shot using Anthropics Haiku 4.5 Model.

```mermaid
sequenceDiagram

    participant Harness
    participant Domain
    participant Agent
    participant DB 

    rect rgb(230, 242, 255)
        note over Harness,DB: Play Hole 1 - Par 4
        Harness->>Agent: GetRecommendation()
        Agent-->>Harness: GetHoleStrategyResponse(club,strategy,confidence)
        Harness->>Agent: RecordShot(result: Fairway)

        Harness->>Agent: GetRecommendation()
        Agent-->>Harness: GetHoleStrategyResponse(club,strategy,confidence)
        Harness->>Agent: RecordShot(result:Green)

        Harness->>Domain: CompleteHole(putts: 2)
    end

    rect rgb(230, 242, 255)
        note over Harness,DB: Play Hole 2 - Par 4
        Harness->>Agent: GetRecommendation()
        Agent-->>Harness: GetHoleStrategyResponse(club,strategy,confidence)
        Harness->>Agent: RecordShot(result: Fairway)

        Harness->>Agent: GetRecommendation()
        Agent-->>Harness: GetHoleStrategyResponse(club,strategy,confidence)
        Harness->>Agent: RecordShot(result:Green)

        Harness->>Domain: CompleteHole(putts: 2)
    end

    rect rgb(230, 242, 255)
        note over Harness,DB: Play Hole 3 - Par 3
        Harness->>Agent: GetRecommendation()
        Agent-->>Harness: GetHoleStrategyResponse(club,strategy,confidence)
        Harness->>Agent: RecordShot(result: Green)

        Harness->>Agent: GetRecommendation()

        Harness->>Domain: CompleteHole(putts: 2)
    end

    Harness->>DB: Persist()

```

The harness was using 37,175 input tokens and producing 1755 output tokens - taking ~30s in total time and costing just under 5c - so just under a cent per recommendation on a clearly cheap model.

| Date | Model | Input | Output | Cache W | Cache R | Time | Cost
| --- | --- | --- | --- | --- | --- | --- 
| Baseline | Haiku | 37175 | 1755 | 0 | 4.6c | 29.8s

What improvements could we get from reducing the round trips and optimising the first call?

## Reduce the Tool Overhead

We clearly still needed the data that the tool calls provides - hole features and historical performance are critical to the agent executing its role.

Our first step was to unwire the tool calls from the agent and restructure our MarkDown context block to include the tool outputs.

In the end the context block looked something like this:

{% gist c73ed2ed609e5536fe855dc4bade4503 ContextBlock.md %}

It's worth calling out that the agent loop still supports the ability to execute tools, we effectively eliminated all round-trips and our Agentic Golf Caddy is now reasoning purely over the prompt and the Context Block shown above.

| Date | Model | Input | Output | Cache W | Cache R | Time | Cost
| --- | --- | --- | --- | --- | --- | --- 
| Baseline | Haiku 4.5      | 37175 | 1755 | 0 | 4.6c | 29.8s
| Tool Removal | Haiku 4.5  | 18534 | 1110 | 0 | 2.4c | | 18.8
| --- | --- | --- | --- | --- | --- | --- 
| Savings |  | 18641 | 645 | | 2.2c |

This change had a significant impact - we reduced the cost per call by almost 50% and improved the latency of the calls by 33% which is obviously a significant improvement.  

## Learnings

This process of measurement and discovery pushed me to more deeply understand concepts that I had a superficial understanding of.  It's easy to be a user of Claude or any other AI harness and talk about things like Prompt Engineering, Context Management or Harness Engineering and sound like you know what you're talking about.

But the reality is that until you dig under the covers and put it in the space of a real-world application it's an academic exercise. 

Everything is a trade-off - every decision that was discussed here had an impact in terms of what the application could do, how the user experienced the services and ultimately the cost of running the system.

In the [first post]({% post_url 2026-06-24-agentic1-fundamentals %}) in this series I spoke about whether an application was truly "Agentic" or simply Generative.  This is another example where we're removing a capability from the agent which would be classified more on the agentic side of the ledger, but in reality we're achieving the same (arguably better) results with a drastically improvement to both cost and latency and at the end of the day thats more important than any particular label.

So the real learning here is to challenge you're assumptions and bias.  Specifically when designing agentic systems, thinking through clearly what the agent needs, when it needs it and how it should be made available is critical.  We understand now the way that agentic loops work and the impact that has on context (and ultimately cost and latency) so we can make better informed decisions and ultimately build better systems.