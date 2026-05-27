---
layout: post-en
title: "AI Produces Garbage?"
date: 2026-05-03
category: en
lang: en
---

*(Translation of the [original French article](/fr/ia-produit-de-la-merde), published in French on [LinkedIn](https://www.linkedin.com/pulse/lia-produit-de-la-merde-frederic-bouchery-t5gbe/) on May 3, 2026.)*

A developer I've shared the floor with at several conferences, and whose opinion I respect, recently commented on [my previous article](/en/ashamed-of-your-code): *"When I use an LLM, it's total garbage. It generates something, and I end up spending more time starting from scratch. It's unbearable."*

I understand the frustration. The current state of generative AI often produces a strange sensation: code that seems to work at first glance but collapses dismally at the first complex business rule. Generated code becomes **disposable**.

But where does that come from? From the tool? No. From the way we use it.

## The Prompt Mirage

Many believed that a magic sentence could replace months of thinking. *"Create me a payment page."* That's a simple prompt. And that's precisely where the trap closes.

An experienced developer knows that the value of their work doesn't lie in writing a for loop. Software development is the management of the implicit. Handling the retry mechanism when the network drops. Thinking about idempotence so you don't charge the customer twice. Anticipating race conditions on inventory. Logging the action correctly for GDPR compliance.

When we reduce our work to surface-level prompting (what's naively called [vibe coding](https://x.com/karpathy/status/1886192184808149383)), we delegate all that critical implicit knowledge to a tool we've told nothing about. AI, left to itself in the face of our description's emptiness, fills the gaps with generic solutions. The result? **Garbage.**

## The Return of Deep Specification

The good old V-model had one cardinal sin: the tunnel effect. You'd specify for months, code for months, and discover the misunderstandings far too late. But its real flaw wasn't the rigor of specification: it was the feedback delay.

With AI, that delay collapses. You specify, the machine produces in minutes, you validate. The tunnel effect disappears. What remains is the V-model's virtue: the requirement for an expressed, framed, and incredibly detailed need.

To drive an AI, you need to define entities, draw the system boundaries, and make every invariant explicit (*"A cart price can never be negative"*).

It's not about writing 300 pages of Word documents that will gather dust. It's about approaching the very essence of our craft: intentional programming.

Back in the nineties, Charles Simonyi was already dreaming of separating Intent (the business "what") from Implementation (the syntax), and that's what he called [Intentional Programming](https://en.wikipedia.org/wiki/Intentional_programming). At the time, programming languages couldn't follow. LLMs have literally just made that possible. AI becomes the universal intention compiler. But it demands that we formulate that intention precisely. That's what I call **intention engineering**.

## The Hyper-Productivity Myth

Which is why I find it remarkable when I hear about that company bragging that its Product Owners can't keep up with demand because developers are going "too fast" thanks to AI. Frankly, that company should be worried. It will bitterly regret that speed the day it discovers the size of the pile it will have to deal with in production.

Replacing the "magic" prompt with intention engineering won't make you code 100 times faster. It may even take **more time than before**.

It would be too easy to turn this into a moralizing lecture: *"If the AI-generated code is bad, it's your fault for specifying poorly."* The reality is crueler. A perfect specification is extraordinarily **expensive**. Pushed to the extreme, flawless specification amounts to nothing less than writing the code mentally. And even with exceptional framing, AI remains fallible: algorithmic hallucination is always lurking.

Instead of typing syntax, you spend time upstream modeling and downstream hunting the machine's inevitable errors. No magic leap.

The real benefit isn't raw speed; it's the **quality of design**. You become infinitely more demanding about architecture. Our craft doesn't disappear; it mutates.

## The Real Cognitive Cost

But here's the thing: if this method works wonders technically, it imposes a colossal hidden cost that few will admit to yet: **cognitive fatigue**.

For decades, typing out the syntax of an algorithm generated a state of [**flow**](https://en.wikipedia.org/wiki/Flow_(psychology)). It was the "Eureka" moment of finding the right abstraction. Today, you no longer type; you **supervise**. AI generates the logic block in 40 seconds. You inspect, you read, you make sure nothing is off.

And reading someone else's code (especially when that someone is a machine) is the most psychologically exhausting task in our craft. Creation generates dopamine. Validation generates brutal [**decision fatigue**](https://en.wikipedia.org/wiki/Ego_depletion). We've gone from being passionate creators to **algorithmic fraud inspectors**. That colleague's frustration comes from this: the raw satisfaction of solving a problem has been replaced by the thankless work of monitoring a machine.

## Pilot, Not Passenger

Does that mean we should throw the tool away? Obviously **not**.

But we have to accept the evolution of the posture. The fatigue I described (this thanklessness of supervising rather than creating) is **real**. It's not a system bug; it's the sign that the craft has changed in nature.

An airline pilot doesn't suffer from not building the plane. They suffer if you take away the controls. Our fight is precisely that: **staying in command**. Setting the course, establishing the constraints, validating what comes out, and taking responsibility for what goes to production.

The real value of a developer in 2026 is no longer mastery of a syntax or framework. It's their capacity to **frame intent precisely** and guarantee the coherence of the system the machine assembled.

The myth of the hyper-productive **code monkey** is over. The era of the **intention architect** is only just beginning.

## And What About Juniors?

That's my real concern. Today, properly piloting AI requires the experience of a senior developer. The result: signals of a junior hiring pullback are multiplying. Logical in the short term. **Catastrophic** in the long term.

Seniors will retire. If we haven't trained a next generation, nobody will know how to design a system without fully delegating it to a machine they no longer understand.

We can't settle for training "prompt monkeys." We need to invent a new trajectory: developers who learn to **design before generating**, to **read before writing**, to anticipate edge cases before validating the machine's code. **Intention architects, trained from the start.**

That's the real work ahead. And it starts now.
