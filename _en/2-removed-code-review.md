---
layout: post-en
title: "Those Who Removed Code Review Are Right"
date: 2026-08-17
category: en
lang: en
published: true
permalink: /en/removed-code-review.html
description: "Developers showed up on my MR with an AI doing the review in their place. The ones who removed code review altogether are almost more honest, because they saw it had lost the race. What remains is knowing what they put in its place, and most put nothing."
---

# Those Who Removed Code Review Are Right
(Published on August 17, 2026 - [Version française](/fr/supprimer-revue-code.html))

They're right about the diagnosis, but for the wrong reasons, and the consequences look nothing like what they imagine.

The shortcut is spreading fast: code comes out in three seconds, so the guardrails start looking like bureaucracy, the PR becomes a brake, peer review a costly courtesy and the integration test a luxury for the timid. We used to say "it compiles, so it works" to mock colleagues who were a little too sure of themselves, but I've met enough of them to know that many sincerely stopped there, and AI has just made their method presentable: it compiles, the generated tests are green, so ship it.

Half of that reasoning deserves to be defended, and the other half to be buried.

## Human review lost the race, and it's arithmetic

An attentive reviewer processes [a few hundred lines per hour](https://smartbear.com/learn/code-review/best-practices-for-peer-code-review/), and beyond that pace their defect detection rate collapses, a limit documented since the [code inspections of the 1970s](https://en.wikipedia.org/wiki/Fagan_inspection) and one that AI has obviously not pushed back. An agent produces that volume in a few seconds, so code now gets written a hundred times faster than it gets read, and no team closes a gap like that by getting better organized.

That leaves only two ways out: either your team actually reads, and review becomes [the new bottleneck](/en/stopped-running-several-agents.html), with PRs rotting for five days while three others go stale behind them, or it pretends. That's where it turns serious, because a rubber-stamp review is worse than no review at all: with no review, everyone knows the risk exists, whereas with an approval you manufacture assurance, and two reviewers scrolling through a 900-line PR in four minutes produce nothing but an audit trail.

I've watched the theater modernize in my own team, where the volume of code produced has exploded while detailed reading dwindled, unable to keep up: developers run an AI to do their review pass, and some show up on my MR with an agent reading in their place. I told them in a meeting that doing the review with an AI means trusting it blindly, and that since it's already the one producing the code, we might as well stop bothering with review at all: it compiles, so it works, straight to prod. What I was holding against them was approving without judging: their agent returns an opinion that changes with every run, nobody answers for it, and the approval still carries the signature of a human who looked at nothing.

Deliberately removing review even has one virtue that the theater will never have: the assumed absence puts the team's back against the wall, and you have no choice but to find the tools and methods that let you ship to production without that phase. Pretending removes that pressure without removing the risk: the code ships unread, and since the box is ticked, nobody will ever build the missing tools.

Those who removed review saw all that, and on the diagnosis, they're right.

## They were wrong about what review actually caught

The mistake isn't the removal, it's the void left behind, and to see it you have to ask honestly what review caught in practice, rather than what it was supposed to catch on paper.

Syntax, style and type errors were automated long before AI, by the compiler, the style checker or static analysis depending on the language. What review caught is a gap in intent, the one that shows up as "wait, why are you going through there?", as "that case, when exactly does it happen?", as "this name promises one thing and the method does another" or as "we decided the opposite three months ago, you were on leave".

None of that can be verified by a compiler, and none of it went away with AI. It's actually worse, since a model produces code that is plausible, well formed, self-consistent and sometimes completely beside the intent, which is exactly the defect profile human review used to catch, and it has become far more frequent. Removing review without putting anything in its place therefore removes the only check aimed at the only thing the machine doesn't verify.

## The compiler doesn't know your business model

The archetypal scenario replays everywhere in different costumes. The agent turns out a refund function with clean code, aligned types and surface tests all green, except it doesn't lock the order's state in the database before calling the payment API. The compiler couldn't care less, neither could the linter, and neither of them knows that refunding the same customer twice is a problem, so the ten minutes of reading you saved will be paid back in table reconciliation and a post-mortem.

None of this condemns AI: the scenario only says where verification needs to aim.

## Continuous deployment is the automation of rigor

Pushing through because it compiles often claims the banner of continuous deployment, and that's a misreading, because the teams that deploy ten times a day didn't remove the checks: they automated all of them, from tests to progressive rollout to rollback. For them, speed and control grew together, the second being the condition of the first, so removing validation in the name of speed means doing far less than everyone else while keeping the vocabulary. Speeding up generation without reinforcing validation mostly shortens the delay between a vague intent and a production incident.

## What must replace review

Manual human blocking really is absurd at this cadence, and those who want to restore the three-hour architecture committee or the PR that waits five days are fighting arithmetic. What replaces review must therefore be automatic, deterministic, and aimed at intent rather than form.

It starts with acceptance criteria derived from a specification written before the code, never after, because a criterion written after the implementation merely describes the implementation. It continues with executable architecture constraints, where the build breaks when a controller reaches straight into the database, because a convention in a wiki blocks nothing while a failing test blocks everything. It runs through business-rule linters, the ones nobody used to write because they cost days of work and now cost a few hours, which probably makes them the most underrated gain in this whole story. And it ends with integration tests that never go back through a model at runtime, because a test that queries an LLM on every run isn't a test, it's an opinion.

What these four bricks have in common is that they return the same verdict on every run, and that none of them depends on the availability of a human at 5 p.m. on a Friday.

## The part that will stay human

Part of the work doesn't automate, and claiming otherwise would be dishonest: deciding whether the feature is the right one, arbitrating an architectural direction or spotting that a spec contradicts another one written six months ago takes a human, and will for a long time. It's just that this human no longer has anything to do at the diff level, because they now work one floor up. Code review survives by changing floors: it becomes specification review, aimed at an artifact that is far shorter and far denser.

Review did something else besides catching defects, too: it trained those who practiced it, in both directions, and that's the only one of its services I have nothing to offer for. Where those who haven't yet put in their years of grease will learn, once nobody reads diffs any more, I don't know.

That validation chain, I built it brick by brick on my own project, and it took me further than I was ready to admit at the start, since today I haven't read 1% of the code that comes out of it. That's the subject of the next article.

Which leaves the uncomfortable question, for those who removed review to move faster: that 900-line read they removed, what did they replace it with? In your team, was review removed, or relocated? If the answer is removed, look for what still verifies intent today, and if you find nothing, you haven't gained velocity, you've just removed the brakes.
