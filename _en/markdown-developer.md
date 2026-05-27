---
layout: post-en
title: "I'm Becoming a Markdown Developer"
date: 2026-05-27
category: en
lang: en
---

I've been coding since 1983. I've seen punch cards, text terminals, and the birth of the Web. Today I keep hearing two arguments that exhaust me. On one side, the vibe coding evangelists who tell you AI is going to replace the tech department. On the other, the purists who refuse to touch AI out of pride or fear that it'll produce garbage.

Both are wrong.

I'm having more fun than ever with AI, but that doesn't mean I've put down my keyboard. My center of gravity has shifted. These days, I spend a lot of time writing specifications, architecture notes, constraints. I write Markdown to drive the agents.

## The value was never in the volume of code produced

Let's be honest: grinding out lines by the mile is the least interesting part of our work. What matters is design, architecture, and system robustness.

I still write code. Often, I'd rather write a complex function or structural change myself, but I no longer do it alone in a corner: I show my code to the AI, explain my approach, my reasoning, and we work from that base. It's a real technical dialogue.

The rest of the time, I treat it as an ultra-fast executor. I design, I set the business invariants, and I delegate the typing. Behind it, I have a merciless critical eye. I review, I validate, and I reject what doesn't work. Sometimes it surprises me with an elegant approach I hadn't thought of. Sometimes it gets its wires crossed and I course-correct.

The danger of vibe coding (throwing a vague prompt and watching the result run without understanding) is that it produces opaque architectures. If your application runs on code that nobody on the team can explain, you no longer own your product — you're its hostage. And when the vendor raises prices by 300% or goes down for a week, what do you do?

## Trust comes from understanding

Trust can only be built on understanding, whether in life or in engineering.

It's no coincidence that an entire field of research (neural network explainability) exists to answer this need: understanding why a model produces certain results, so you can trust it or adjust it.

I'm a PHP developer. When AI generates PHP, I dissect it, I question it, I reject it if necessary. But when it produces JavaScript or CSS (languages I know but don't master), I tend to say "sure, if you say so." And honestly, in that case, I have no idea if that code respects the standards: I make a conscious cost/risk tradeoff. If the impact is limited enough, I accept it as is. Otherwise, I delegate the review to whoever can judge it: my JS expert for JavaScript, my front-end developer for CSS.

The level of trust you give to generated code must be proportional to your ability to critique it. AI must conform to sound architectural principles so the system remains intelligible to a human, not the other way around.

## The return of specification

The V-model had its flaws (it was heavy, tedious, top-down), but it had a rigour we abandoned too quickly. Roles were clearly separated: the business side wrote the functional specifications (the "what"), the technical team translated them into technical specifications (the "how"), and developers arrived at the end of a long tunnel. Acceptance tests validated conformance to the original requirements. At least, we tried.

Agile simplified all of that. The Product Owner expresses a requirement, often incomplete (e.g., "the user must be authenticated," with no further detail), and developers fill in the gaps. We gained velocity, but lost the rigour of analysis.

You could argue that user stories, with their acceptance criteria, are already a form of functional testing. That's true, but they remain incomplete, written in natural language, and above all non-executable. The developer had to interpret them, complete them, and translate them into real tests, and most often didn't, for lack of time. Agile had the right intuition and stopped halfway.

With AI, we can rehabilitate that rigour, but lighter, iterative, carried by the developer themselves. Markdown has become the natural language of this practice: it's the format of documentation, it's readable by AI, it's versionable. AI participates in refining specs through dialogue, exactly like a technical conversation with a colleague. In a previous article, I talked about intentional programming, and Markdown writing is now its natural language.

Be careful, though, of one trap: if you write specs alone in a corner with AI, you can fall into an analysis tunnel instead of an implementation tunnel. AI moves fast, and you can very quickly produce a very precise spec... for the wrong thing.

This is where acceptance tests become a communication tool, not just a validation tool. You identify the gaps in the PO's requirements ("if I implement authentication without a password, does that work for you?"), you propose, you discuss. Then you have the PO review a selection of tests to confirm you understood the essentials. You're not going to have them review every single test, that would be tedious, and the PO's job isn't to dive into implementation details. You filter by the two registers inherited from the V-model: the functional behaviour of the system (the "what") you have the PO validate, because that's their domain, and the technical choices (the "how": architecture, database engine, layer structure) you validate yourself as architect, because that's your responsibility.

This is exactly where trust is built between a PO and an architect. The PO delegates because they know you can distinguish what deserves their decision from what you can settle yourself. If you understand a bit of their business domain, you can make certain decisions for them. That judgment is what makes an architect valuable.

This is no longer documentation after the fact; it's steering.

## Tests: from safety net to steering tool

Tests have always had two roles, but practically everyone only identifies one.

The first, everyone knows: regression. The second is less often articulated: design. Writing a test before the code forces you to design the API, the inputs, the outputs, the DX. You decide how that function will be used before writing a single line. Double effect: the resulting code is naturally more testable, less dependent on mocks, because the testability constraint was set from the start.

Choosing the testing tool (PHPUnit, Playwright, Selenium, JMeter) forces you to answer a precise question: what constitutes proof that it works? That clarity directly benefits maintenance and evolution. In six months, the test still says what the system is supposed to do, and it remains executable.

With AI, a third role emerges: verification of conformance to the spec. Without tests, how do you know if the AI followed every point in your specification? If it quietly hallucinated on an edge case? You can't mentally review every line and compare it to the requirements. Acceptance tests answer that question objectively: pass or fail, no room for interpretation or silent omissions.

Tests crystallize the requirement: you don't write them to verify existing code, but to define what you want to build, and the AI then codes to make them pass. It's TDD, shifted up one level of abstraction. In refactoring or maintenance, the stakes are different.

Every piece of software has a healthy lifespan. Without AI, many projects reach that inflection point within a few years. With vibe coding, it can come much sooner: the first change request sometimes reveals an architecture that's impossible to evolve.

With well-guided AI (precise specs, solid tests), you at least recover what you had before, and you unlock something that was previously economic suicide: the big rewrite becomes viable. Switching from MongoDB back to PostgreSQL is doable when you have something to automatically verify that nothing broke.

One important nuance: for certain critical migrations, I don't trust the LLM to apply changes directly, so I ask it to write the migration script or tool, so the strategy stays under my control.

## Moving up one level of abstraction

The developer's profession doesn't die; it changes nature.

We are leaving the era of syntax for the era of conceptual clarity. If you don't know what you want to build, if you don't master your business rules and your architectural patterns, AI is just going to help you produce bad quality ten times faster.

I sometimes hear companies say they no longer need developers because the PO will delegate directly to AI, which makes me smile. Product Owners are not equipped to fill all the technical gaps in a spec, and they will inevitably delegate to someone who can bridge the gap between business requirements and what the machine can execute.

That role is the architect's. The chain taking shape looks like this:

> Product Owner > Architect (AI-boosted) > AI Agent(s)

Part of the job shifts from syntactic production to systems design. It requires more rigour, not less. We write just as much, but it's no longer parentheses and indentation; it's long sentences and arguments. This isn't a promise of velocity; it's a promise of quality.

But this move up the abstraction ladder isn't available at every point in a career. That instinct builds up over years of writing code, making architectural mistakes, debugging at night systems you designed badly: the ability to frame intent precisely, to sense a missing invariant, to spot where the AI goes off the rails on an edge case. A developer who delegates heavily to AI before acquiring those reflexes isn't moving up in abstraction; they're short-circuiting the phase where judgment is formed. That's what worries me most about where the profession is heading, far more than the posture of senior developers.

Yes, Markdown has become my main steering language, but I keep my hands dirty whenever the situation calls for it, and strangely, I've never felt more like I'm doing my actual job as a developer.

---

The real skill of this era isn't knowing how to talk to machines; it's having something to say to them.
