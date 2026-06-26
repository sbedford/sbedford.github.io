---
layout: post
title: Welcome back - let's build something?
byline: Resurrecting this blog to document a learning journey on Agentic systems design
banner: /images/2026-06-11-welcome-back-banner.svg
series: Building an Agentic Golf Caddie
---

It's been a while since my last post...whats 9 years between friends hey?

A lot has happened over that time - I left consulting, joined Google, became a manager, left Google, joined HashiCorp, moved from supporting Australia to building a team across Asia Pacific and Japan...and thats just the on the professional front.

At heart, I'm still building.  That's what I do, but the focus has shifted. 

I've gone from building systems and applications to building teams, building technical sales strategies, building customer retention strategies but you know what? 

While I love my teams and what I'm doing, there's a strong part of me that really misses what made me fall in love with tech in the first place.

Building applications, solving problems - everyone loves a spreadsheet but there's more to life!

So with that in mind, this series is my attempt to learn something new.   

I've got a pretty good conceptual understanding of Artificial Intelligence and Generative AI but theres a huge difference between knowing how things work at a conceptual level and actually building something on it.

# The Problem Statement

I've always learned better by doing that reading - and its always easier to build something if you can apply it to something that your genuinely passionate about.

I'm a very keen golfer and play at least once each weekend in my home club's competition.  Now golf can be an incredibly analytical sport.  You can absolutely map your improvement in a number of key areas directly to your overall performance - for example, reduce the number of putts per round or hit more fairways or greens and you will see your handicap improve.

I track a lot of data on my performance each week and track trends and statistics to see how my game is trending and where there are areas to improve.

However, this is looking at the event after its happened when what I would really love is real-time guidance mid-round.  Consistently hitting it into trouble on a particular hole? Advise me to club down short of the trouble. Consistently miss short on an approach shot early in my round? Remind me to take an extra club.

So this is the problem we're going to try and solve - and we'll need to solve a bunch of problems along the way with respect to data modelling, model development, overall application architecture which is exactly what I need to get to grips with AI in a real-world scenario.

# The Thesis

This problem space is interesting.  In many ways I can see how we would solve this using a deterministic mode using traditional data analysis - particularly with the data that I have available right now in the spreadsheet.  However, as we get more advanced and include more real-time data points - weather, in-round performance, contextual guidance (e.g. your in the trees with a terrible lie) it moves into a world where the ability to use data to inform but reason over it becomes much more impactful.

We have a long way to go before we get to our final destination here, and this journey will be full of mis-steps im sure, but ultimately our goal is to use our past performance to inform our next action. **Classic AI use-case.**

# The Plan

The architecture is still fairly rough but this (thanks Claude) is a decent place to start.

![The Agentic Golf Caddie Archiecture](/images/2026-06-11-golf-caddie-architecture.svg "The Agentic Golf Caddie Archiecture")

We'll need to make some decisions on the way and some areas might be best handled in a traditional or non-agentic fashion but for the purposes of the exercise, we'll default to Agentic to start.

The broad plan of attack for the project is below

| # | Activity | Outcomes |
|---|----------|----------|
| 1 | Basic Data Model | A relationship representation of the tracking spreadsheet we can build around | 
| 2 | Agent Tool Design | A streamlined set of tools the agent can use to interrogate the data |
| 3 | Round Analysis Agent | Build a baseline agent that uses the tools to analyse history and provide a set of hole by hole pre-round recommendations |
| 4 | Quality Assurance | Baseline performance. Identify and implement areas to improve recommendations |

Once we get to this point, we'll re-asses the plan and explore real-time data integrations, application build for data capture and recommendation surfacing, etc.

Lets get building!