---
layout: post
title: The agent didn't lie, it just didnt know
byline: Garbage in, confidence out.
featured: true
banner: /images/2026-06-26-agentic2-banner.svg
series: Building an Agentic Golf Caddie
---

I'm standing on the tee on the 6th hole at St Michaels Golf Course.

Par 5, around 500m from the back tees.

Easy driver every day of the week - there is trouble on the right, a few bunkers on the left but plenty of fairway further left coming back up on the 8th.  

Not the ideal line, but still not something to worry about.

I whip out my Agentic Golf Caddy and ask for a recommendation....

>Par 5, stroke index 16 (no shot received). Sean misses left consistently but fairway bunkers 159-205m out on left are avoidable at 170m carry. Water right 52-210m from green eliminates driver/3-hybrid gamble. 5-iron leaves 331m to green for second shot—a comfortable 3-wood or hybrid layup. Historical data shows 60% fairway miss rate; conservative positioning avoids double.

Hit 5-iron short of the bunker? What the hell?  Are you crazy!

## Hallucination or Just Bad Data?

Before we go further, let's get clear on what a hallucination actually is.

[Google](https://cloud.google.com/discover/what-are-ai-hallucinations) provides a fairly narrow, model-centric view of a hallucination.

>AI hallucinations are incorrect or misleading results that AI models generate. These errors can be caused by a variety of factors, including insufficient training data, incorrect assumptions made by the model, or biases in the data used to train the model. AI hallucinations can be a problem for AI systems that are used to make important decisions, such as medical diagnoses or financial trading.

However, in agentic systems there's a whole other class of problem - Agents can only reason over what you give them. 

Gaps in context, whether in the pre-built context block or data returned through tool use, don't produce errors. 

__They produce confident answers based on a slightly different world than the one you're actually standing in.__

## What's in the Agent's world?

Our Agentic Golf Caddy application has a data model that captures key information about the Golfer, the Course and, most importantly, the Holes that the golfer is playing on.

The agent's view of the world comes through three channels and it only knows what we tell it: context, tooling and the prompt.

### Context block

* Golfer Details - Handicap
* Course Details - Course ID, Tee's
* Round Details - Holes Played, Performance to Par
* Club Details - Reliable carry distance, dominant miss 

### Tool Definitions

__get_hole_stats__

Provides a data dump of all the rounds where the Golfer has played this hole in the last 6 months. 

* What did they score? 
* Did they hit the fairway? 
* Where did they miss? 
* How many putts were taken, etc.

![get_hole_stats](/images/2026-06-26-agentic2-get-hole-stats.png "get_hole_stats")

__get_hole_layout__

Provides information about the hole and tee configuration.  

* Par
* Stroke Index
* Distance
* And most importantly features which describe elements of the hole that the agent should consider when providing a recommendation.

![get_hole_layout](/images/2026-06-26-agentic2-get-hole-layout.png "get_hole_layout")

### Prompt

> Sean Bedford is playing shot 1 on Hole 6 at St Michaels Golf Course. Standing on the tee 501m from the pin to a front_centre pin. Provide a strategy

## So Why am I hitting 5-Iron?

Before we can answer that question we need to look at the data that the model has to reason over.

The agent has been provided a lot of information on where the player is and the distance of the hole, so thats not the problem.

The reasoning provided in the response specifically calls out the bunkers which is described in the get_hole_layout tool response:

| Type | Side | Reference Point | DistanceStart | DistanceEnd | Label |
|---|---|---|---|---|---|
| bailout | right | green | 0 | 21 | Bailout area 21m short right of green |
| bunker | left | green | 173 | 159 | Fairway bunker 159-173m from green left |
| bunker | left | green | 205 | 192 | Fairway bunker 192-205m from green left |
| bunker | left | green | 260 | 250 | Fairway bunker 250-260m from green left |
| dense_scrub | right | green | 339 | 227 | Dense scrub right from 227m to 339m from the green |
| out_of_bounds | long | green | 25 | 0 | Out of bounds 25m from centre of green |
| water | right | green | 210 | 52 | Water starts 210m from green till 52m out |

Can you spot the problem?  The issue is not what is in the data, but what isnt!

In this case, the model had such a narrow view of the hole, its recommendations were correct from its point of view.  

The agent just didnt have a complete picture of the hole, where the safe miss was, etc because I hadn't described it accurately enough. 

For the technically curious, this sits closer to what researchers call an extrinsic hallucination — the model generating output that can't be grounded in source context because the source context was incomplete.

But in the real world it completely destroys the trust in the system.

It also highlights a huge flaw in the way I was modelling the data - manually capturing features of the hole like this will both naturally skew the model in a certain direction and, at the same time, limit its view to only those "hazards" called out.

If I was to add an item to the database describing the width of the fairway, or that there was a safe bailout long left, the model would most likely have told me to hit driver and move on. 

Now that's definitely a fix, but it wont address the root cause or stop this from happening again.

## What's the long term solution?

The decision to capture our "feature" dataset in this way was a deliberate call and I knew it had limits when I designed it.

But building a long-term solution meant that I needed to change my approach.

### Change #1 - From Distance to Real World Coordinates

Firstly, we need to change from operating using distances - in our prompt, in our context and in our features data model.  

We need to capture the players current GPS position and then analyse the hole features that are in play by looking at polygon data.

### Change #2 - Real-World Mapped Golf Courses

We're currently manually describing the features of the hole working backwards from a reference point.

Now this approach is clearly limited - not just because of the effort required to catalog a hole accurately but also in providing a big picture, geometric view of the physical surrounds.

All holes are treated independently with no relationship between them and this really compounds the data capture granularity issues we're seeing in this situation.

What I need is a set of proper golf course geography data - services like the [Golf API](https://golfapi.io/) could be used to outsource the capture  of the bounding boxes.

### The interesting impact

Feeding raw polygon data into the model and asking it to work it out would be a horrible use of tokens.

Beyond the token cost, large language models are poor at reasoning over geospatial data - they're just numbers and the context is hidden in the detail.

So what's the solution? Build a translation layer in Go.

Retrieve the features, run the geospatial calculations based on the player's current location and club-carry distances, and cover both the current hole and the surrounds. 

The get_hole_layout tool stays, it just gets smarter.

## Wrap-up

Back on the tee, the agent wasn't broken.

I told it where I was and it looked at the data - water on the right, bunkers on the left and a 60% fairway miss-rate and recommended the safe play.

Based on all the information it had, that was the right call.

But the problem was in what the agent couldn't see - the context that not only wasn't shared - it wasn't captured!

This highlights a key trade-off in designing this system that I'll have to put up with for now but have a long-term strategy to solve.

Remember, confident but wrong is a dangerous failure mode.