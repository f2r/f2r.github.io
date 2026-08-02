---
layout: post-en
title: "I Stopped Running Several Agents"
date: 2026-08-02
category: en
lang: en
permalink: /en/stopped-running-several-agents.html
description: "Waiting isn't wasting. I was filling every second of generation time by opening one more agent, until I became the single point every one of their questions had to pass through. What it cost me, and what I decided to stop."
---

# I Stopped Running Several Agents
(Published on August 2, 2026 - [Version française](/fr/arrete-plusieurs-agents))

Yesterday afternoon I had four agents working for me and I spent my time answering their questions, until the point where I no longer knew which one was waiting for what.

There's nothing improvised about the way I work with AI, and I described it in detail the day [I became a Markdown developer](/en/markdown-developer). I now spend most of my time writing specifications, I have acceptance tests written to match those specifications, and I only launch the implementation in full autonomy once those two steps hold. The agent then runs for a few minutes, sometimes a good deal longer, without needing me, except that I find myself in front of a running terminal twiddling my thumbs, and the reflex that follows is to open another agent on another task, then another.

Nobody asked me to do that, and that's exactly what bothers me, because this drift isn't the price of bad framing: it feeds on the free time that good framing gives me back.

The most embarrassing part is that I know the theory on limiting work in progress ([Little's law](https://en.wikipedia.org/wiki/Little%27s_law)), and "stop starting, start finishing" has been around for as long as I can remember. I've simply never been good at applying any of it, ticket management has never been my strong suit, and it only took agents arriving in my shell for that weakness to start costing me.

What happens next is mechanical, since every agent eventually hits a decision it can't make alone, so it stops and it asks me a question. With four agents I find myself on call without having decided to be, working through questions one at a time and never seeing the queue empty.

What drained me is that continuous pressure of four of them waiting on an answer from me to move forward, a load I would never have accepted from a human team.

## The bottleneck had moved without me noticing

The [theory of constraints](https://en.wikipedia.org/wiki/Theory_of_constraints) is merciless: optimizing a step upstream of the bottleneck adds nothing at the far end, it just piles up inventory in front of the constraint.

In a furniture workshop where the cutting station turns out 100 planks an hour, hand sanding handles 10 and painting absorbs 100, production tops out at 10 pieces of furniture an hour, and that physical reality couldn't care less about the two fast stations. If I buy a laser cutter that climbs to 1,000 planks an hour, the output doesn't budge by a single piece of furniture, but the workshop ends up drowning under 990 planks that fill the aisles and warp while they wait their turn. That is exactly what inventory in front of the constraint looks like.

## In software, the inventory is invisible

Nobody trips over the planks. All my life my constraint was writing, meaning typing syntax and churning out code, and AI crushed that problem, so that 500 lines now cost me nothing at all to produce. The constraint immediately jumped to the only resource that doesn't scale, namely me, with my capacity to read and to decide.

So generating ten times faster unblocks nothing, it just piles up stock in front of me: open branches everywhere, PRs rotting away and a whole day spent arbitrating proposals I don't have time to understand. I've built a traffic jam and I'm its only junction.

When you force-feed a station that's already saturated, you trigger a failure: on a server the requests start timing out, and on a human it's called [decision fatigue](/en/ai-produces-garbage). I wrote about it back when I pointed out that reading code produced by a machine is the most exhausting task in this profession, and it ends in bugs nobody catches any more.

## Four agents all routing back through me

Adding developers to a late project makes it later still, everyone knows [Brooks's law](https://en.wikipedia.org/wiki/Brooks%27s_law), and the mechanism is simple: coordination cost climbs faster than production. Except that with four agents running in parallel, part of the work inevitably overlaps, and every conflict that needs a decision routes back to me, which makes me the data bus of my own augmented team.

While one refactors the authentication module, another writes a component against the old signature and the third pulls in a dependency that breaks the first one's types. Since they all work in the same repository, each of them ends up noticing files it never touched. Git knows how to merge text and the agents recover on their own most of the time, but every recovery gives them one more reason to come and ask me what I think, and that's how the coordination cost gets paid in interruptions.

Running them on separate projects removes the conflicts without fixing the problem, because every question then forces me to reload an entire project into my head: on a single repository I mix things up, and on separate repositories every decision gets slower. Either way, everything routes back to the same brain.

## The time they give back arrives in crumbs

Four agents don't free up a clean two-hour block, they spit out forty windows of three minutes each.

And you don't design an architecture in 180-second slices, any more than you keep up with the industry in them, and chopped-up time produces nothing but scrolling. That's why the list of good intentions never holds, the one where you tell yourself "while the agent works, I'll answer my 548 unread emails, I'll read up and I'll update the board", because it assumes continuous stretches of time and four parallel agents leave me none.

The nuance is decisive, because the length of those stretches depends entirely on how I carved up the work: a single task, specified and covered by its acceptance tests, gives me that time in one block, whereas four tasks launched in parallel shred exactly the same duration into confetti. The problem was never the total duration, it's the size of the largest continuous piece.

Beware too of doing code review during that time, because reviewing the code of one task while an agent produces another is still context switching, just with one agent fewer. It draws on the same resource as everything else: my capacity to read and to decide. Code review is development.

## Stepping in right now costs three seconds

Filling every second of generation by opening another task is a misplaced managerial obsession with output, the same one that blames an operator for standing in front of a machine that's running.

Following the agent on the task I've just handed it is another matter, since I stay in the same context, and it's even the only activity that genuinely fits in a three-minute window. That's where I see the absurdity live, when the agent heads down a dead end, when it imports a 50 MB package for a single utility function or when it quietly slips in a shaky mock to turn a test green.

When I'm following what the agent does, I sometimes interrupt it mid-flight because I can see it heading in the wrong direction, whereas with several agents I scroll back up through the work log and discover after the fact that it wasted time for nothing. Walking it back doesn't take long, but everything it produced in the meantime goes in the bin, when stepping in live would have cost me three seconds. And you still have to be looking at the right moment, because my agent displays its reasoning and its tool calls while it works, then folds everything away once the task is done: to notice it went wrong, you would have to reopen the reasoning thread, which you practically never do, and the drift nobody saw gets paid for much later, as a bug. That's precisely the [vigilance](/en/ashamed-of-your-code) I described as having nothing to do with paranoid distrust, since it's the normal work of someone who understands what they commit. That still doesn't make watching the agent a way to spend a day, because those drifts belong to my validation chain, meaning the acceptance tests and all the automated checks that block a delivery without my sign-off.

## Specify bigger, don't prompt faster

Spending your day glued to the terminal validating thirty-second tasks is agent babysitting, and the leverage is somewhere else, in the size of the scope you delegate.

A badly framed two-minute task forces me to stay in front of it, whereas a task with a wide scope, with clear constraints and acceptance criteria written beforehand, gives me a continuous block. The entry cost is real, because that task has to be written first, and that's exactly the work we run away from when we launch four agents on four vague requests.

## The day multi-agent holds up

When I described [multi-agent architecture](/en/llm-probability-orchestration), the conductor was a process delegating to specialized, independent agents, and that's precisely where I went off course, since in my terminal that conductor was me, by hand. Multi-agent will hold up the day it no longer needs me to validate, and it collapses as long as I remain the only loop. When my validation chain rejects the drifts without going through my brain, the bottleneck will move again and parallelism will become a defensible option once more, so the right question is about the load it absorbs without me.

Those who keep ten agents running without ending up fried have no more stamina than I do, they just have a better validation chain, and that's another subject.

In the meantime I've stopped running several of them, and I won't pretend it's comfortable, because the temptation comes back every time an agent goes off for a quarter of an hour and a terminal running while I do nothing looks like waste.

Waiting isn't wasting. My inability to tolerate visible idleness is the real cause of my fatigue.

Except that resisting doesn't make the emptiness disappear, it only makes it available, and it has to be filled with something: putting the project in order, writing down what nobody has written down, settling what I kept postponing. Those are exactly the tasks I've never been good at, and I no longer have the excuse of not having the time to keep avoiding them.

The theory of constraints calls this elevating the constraint, and it's the answer to the objection about starving the bottleneck: the time I no longer spend working through questions goes into increasing the capacity of the only workstation in the shop that money can't buy.

Whether I'll stick to it, or find myself a good reason to open a fifth terminal, remains to be seen.
