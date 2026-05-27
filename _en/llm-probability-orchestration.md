---
layout: post-en
title: "LLM: From Probability to Orchestration"
date: 2026-03-04
category: en
lang: en
image: /img/llm-probabilite-orchestration.png
description: "What you think you know about AI is half the picture. The real revolution isn't in the models — it's in how we orchestrate them. From tokens to multi-agent systems: how we went from word prediction to distributed systems engineering in two years."
---

# LLM: From Probability to Orchestration
(Published on March 4, 2026 - [Version française](/fr/llm-probabilite-orchestration))

> **Disclaimer:** Written in March 2026, this article covers the field of generative artificial intelligence, a constantly evolving sector. The concepts and architectures presented here are based on knowledge and developments at that date and are subject to rapid change. These reflections should be considered a temporary snapshot, not definitive conclusions.

If you ask a language model what "two plus two" equals, it doesn't count. It doesn't invoke an arithmetic processor; it runs a neural network predicting a structured response pattern. Although this response has been refined through [human-guided reinforcement learning](https://en.wikipedia.org/wiki/Reinforcement_learning_from_human_feedback) to ensure accuracy, the underlying operation remains stochastic inference, not traditional arithmetic.

This is the foundational premise of all modern generative AI: a large language model (LLM) is, at its core, an autoregressive text prediction engine. Yet between this rudimentary probabilistic oracle and the complex software systems deployed today in production under the name multi-agent architectures, there is a vast technological gulf. We crossed this gap in record time, moving from engineering isolated neural networks to designing complex distributed systems.

In this article, I want to break down this rapid evolution.

## Tokens and the Context Window

To understand the internal mechanics and physical limitations of an LLM, two foundational concepts need demystifying: the token and the context window.

A computer doesn't read words; it manipulates numbers. Before a sentence reaches the neural network, it goes through a tokenization step. Raw text is split into fundamental units called tokens. A token isn't necessarily a full word: depending on complexity, it might be a common word, a syllable, a prefix, or even a single character or punctuation mark. The most common method today is the [BPE (Byte-Pair Encoding)](https://en.wikipedia.org/wiki/Byte-pair_encoding) algorithm. Its role is to compress text intelligently by iteratively merging character sequences that appear most often together in human language. A rare word might therefore be broken into three small tokens, while a very common word gets encoded as a single numeric token, reducing computational load.

The context window is the AI's short-term memory for a given conversation. It's the hard limit, measured in tokens, of everything the model can "consider" simultaneously when generating the next token. This window includes your initial question, the conversation history, and the response the model is currently writing. Asking an LLM to read text that exceeds its context window is like asking someone to memorize a 10,000-digit phone number: whatever overflows is simply discarded by the underlying [Transformer](https://en.wikipedia.org/wiki/Transformer_(deep_learning_architecture)) architecture.

Before entering the neural network, this sequence of tokens passes through a transformation step that uses an attention mechanism to link tokens together and position them within the full context. This allows the neural network to process tokens relative to each other, not in isolation. In the sentence "The white cat is lying on the green sofa," "lying" and "white" refer to "cat," while "green" refers to "sofa." As far as anyone knows, green cats are rare, and sofas seldom lie on top of cats.

## Breaking Out of Amnesia

In the early days, using an LLM was like querying a closed monolithic system. The software architecture amounted to sending a string (the prompt) to a neural network, which returned another string. That approach, however impressive at mimicking natural language, had major structural flaws for any application requiring precision and up-to-date information.

The first flaw is hallucination, which isn't a software bug but an intrinsic feature of the attention algorithm. If the model doesn't have the answer in its weights, it is nevertheless constrained to maximize the probability of the next sequence. It generates the most semantically plausible token, even if that means fabricating facts entirely, because the model optimizes for linguistic coherence, not factual truth.

The second flaw is knowledge obsolescence. A model's knowledge is frozen at its pre-training cutoff date. To bring it up to date initially required full retraining, an expensive, slow operation that was often inefficient for simply injecting new facts.

To address these limitations, the field quickly recognized that models shouldn't be treated as databases but as reasoning engines that need to be supplied with external data. This realization triggered an intense technological race between two distinct approaches to managing external context: massively expanding context windows and developing RAG (Retrieval Augmented Generation).

With a very large context window, up to 2 million tokens (roughly three complete books), you can inject the entire day's news articles and say "Using only the information above, answer my question." RAG, by contrast, connects the model to a search engine to filter an external database and injects into the context only the precise excerpts needed to answer the question.

So why call it a technological race? Because it's really a philosophical and technical battle:

- Long Context advocates (like Gemini 1.5 Pro) argue: "Stop tinkering with external search engines (RAG) and just give the model the ability to read everything at once."
- RAG advocates counter: "Reading everything every time is wasteful; targeted retrieval is more precise and economical."

## The Infinite Context Illusion

Throughout 2024 and 2025, the industry witnessed a spectacular, almost absurd, expansion in context window sizes. Vendors engaged in a marketing arms race, offering models capable of ingesting hundreds of thousands, then millions of tokens in a single pass. Google's Gemini ecosystem pushed these limits, with Gemini 1.5 Pro and Gemini 2.0 Flash accepting up to two million and one million tokens respectively. The experimental Gemini 3 Pro maintains a one-million-token window while extending multimodal support to handle text, images, video, and audio streams simultaneously.

From a purely theoretical standpoint, this capability raised a seductive and often-repeated hypothesis in developer circles: if a model can now ingest an entire software reference manual, a legal contract database, or a project's complete history in a single request, does RAG become obsolete? Why maintain vector databases, compute embeddings, and tune chunking algorithms when you can simply send everything to the model raw?

However, real-world experience and research quickly demonstrated that dumping massive amounts of text into a context window runs headlong into hard constraints: attention degradation, cost, and latency.

## The Central Memory Hole Syndrome

The most notable constraint with giant context windows is a fundamental limitation of the attention algorithm: the ["Lost in the Middle"](https://arxiv.org/abs/2307.03172) phenomenon.

Research from Stanford, UC Berkeley, and Samaya AI revealed that when a model processes an extremely long context, its ability to retrieve specific information depends heavily on where that information is located. Models show very high accuracy when relevant information sits at the very beginning or end of the prompt. But when data is buried in the middle of a long context, the model's ability to retrieve and use it can drop significantly. The model starts hallucinating or simply forgets crucial details buried in the mass, because its autoregressive design gives the system a bias toward the earliest elements of an input, which are accessed and re-evaluated repeatedly with each new token. As the model deepens and attention layers stack up, this bias amplifies disproportionately.

As of 2026, while complex positional encodings (such as [Rotary Position Embedding](https://arxiv.org/abs/2104.09864)) have been developed to mathematically link words to their immediate neighbors, the effect is still partially present on million-token sequences, and models still create an informational blur zone in the middle of a massive context.

## The Cost and Latency Wall

Beyond the loss of algorithmic precision, processing an extended context demands massive computational resources (RAM, GPU, TPU) that grow exponentially with sequence length.

The approach of inserting entire documents into the context causes inference costs to explode, since virtually all API providers charge per token ingested.

Even if "lite" or "flash" models offer high-throughput, low-cost processing, sending half a million tokens with every user interaction for a simple question multiplies operational costs irrationally compared to targeted retrieval of a few hundred tokens. Models capable of ingesting giant contexts often impose premium pricing that grows along two dimensions: token volume and per-token price.

Latency also becomes a barrier for user experience and applications requiring near-real-time responses. The time needed for the neural network to compute attention matrices over a multi-million-token sequence (Time to First Token) increases considerably. Comparative benchmarks show that median latency for analyzing a saturated context on a heavy model can be more than double that of a lightweight model processing a restricted context. In high-traffic production scenarios, the giant context approach collapses under its own financial and computational weight.

## The Maturation of RAG

Faced with the financial and technical challenges of "infinite context," RAG didn't disappear. On the contrary, it established itself as the fastest, most economical method offering the best precision control for integrating private data.

Technological evolution has, however, transformed RAG. It has moved well beyond its naive early stage of simply chunking text, converting to vectors, and running similarity searches.

Today, RAG architecture has become a true context engine, forming the unified information management layer of any serious AI application. Conversation history management (memory) is itself just a specific instantiation of RAG, where the data source is the interaction log and temporal relevance takes priority.

A 2026 RAG implementation relies on complex pipelines designed to maximize precision while minimizing the number of tokens sent to the context. Retrieval happens in two phases.

The first phase performs a hybrid search, similar to early RAG methods, combining vector search with traditional weighted keyword matching algorithms like [BM25](https://en.wikipedia.org/wiki/Okapi_BM25).

Once a set of potentially relevant documents has been retrieved, the second phase kicks in: reranking via a [Cross-Encoder](https://sbert.net/examples/sentence_transformer/applications/retrieve_rerank/README.html) that evaluates the query and document simultaneously, offering far superior semantic matching precision to determine the true relevance of text fragments.

This funnel strategy ruthlessly filters out noise. Instead of sending a million tokens to the LLM, the RAG system transmits only the most relevant document portions.

More importantly, the orchestrator performs strategic ordering: it deliberately places the most critical elements at the very beginning and end of the sequence injected into the prompt. The system deliberately exploits the attention mechanism's positional bias to ensure the language model doesn't overlook any vital information when generating its response. It's an approach that avoids drowning the model.

The challenge is no longer to pit RAG against extended context but to merge them. Modern RAG acts as an intelligent filter: it doesn't just reduce data volume, it pre-digests and structures information so the language model can exploit its context window, even a large one, with greater efficiency.

## When the Model Becomes the Conductor

A model's ability to ingest the right context via RAG remains, fundamentally, a passive system. A language model that merely reads and synthesizes is confined to a text bubble. The real shift happened when the development community decided to give these isolated brains "hands" through function calling.

Function calling changes the fundamental nature of the interaction. The model no longer merely predicts response text; it autonomously decides to interrupt its text generation to request the execution of an external action.

During its sequential inference, if the model deduces from the conversation that the user needs a complex calculation, it no longer tries to guess the result statistically (which would inevitably lead to a mathematical hallucination), but instead triggers a Python script whose result is re-injected as new data into the context. The LLM then resumes its inference, reads the result of its own action, and generates the natural language summary.

This mechanism transforms the language model. It stops statistically guessing answers and becomes a new kind of control center, capable of interacting with its host environment. However, entrusting decision-making to a standard model designed to generate fluent text quickly shows its limits when the action process requires multi-step planning, conditional logic, or self-correction. This is where the second major development of recent years comes in: the emergence of analytical inference.

## Learning to Think Before Speaking

For years, AI development was governed by neural network pre-training. The assumption was simple and blunt: to increase a model's intelligence and precision, you just had to massively scale the dataset size, the number of neural network parameters, and the compute allocated during the initial months of training. Once trained, models execute queries quickly, linearly, and deterministically.

In cognitive science, this mode of operation is called "automatic processing" in humans (also known as ["System 1"](https://en.wikipedia.org/wiki/Thinking,_Fast_and_Slow) or "heuristic processing"): a fast, automatic, reactive, and highly contextualized mode of thinking. It's what we call "intuition," which in LLMs translates to "statistical intuition." For translation, summarization, or fluid conversational chat, this system is perfect. But faced with complex software development problems, advanced mathematics, or logic puzzles requiring tree-structured planning, this fast thinking, incapable of anticipation or backtracking, fails dismally.

The year 2024, maturing through 2025-2026, marked a fundamental break with the introduction of reasoning models. This resembles the human thinking mode called "controlled processing" (also known as ["System 2"](https://en.wikipedia.org/wiki/Thinking,_Fast_and_Slow) or "analytical processing"): a slower, energy-intensive, conscious, and deliberate mode of thought.

To illustrate this break pragmatically, consider the analogy of a child learning mathematics. When asked "what's 7 times 8," they answer "56" instantly. They're not really calculating; they're retrieving memorized information through a simple, automatic, unconscious reflexive association. That's automatic processing. But if you ask them to multiply "342 by 87," intuition isn't enough. The child needs a scratchpad, has to set up the operation, calculate the units, manage the carries, then sum the sub-totals step by step. This laborious, sequential, logical, and conscious process is controlled processing. Modern reasoning models replicate this behavior exactly: they deploy a visible internal scratchpad to work through the problem before committing to an answer. Think seven times before you speak.

When a user submits a complex problem, a reasoning model doesn't immediately write the response. It activates a ["chain of thought"](https://en.wikipedia.org/wiki/Prompt_engineering#Chain-of-thought) to explore different branches of reasoning, generate hypotheses, evaluate them dynamically, recognize its own logical dead ends, abandon lines of inquiry, and self-correct. This iterative process unfolds internally, invisible to the user, and consumes thousands of tokens before producing the final output.

## The Limits of Reasoning

However, this spectacular gain in pure logic and autonomous decision-making capacity comes with trade-offs in raw performance and financial costs. Using a chain of thought causes response generation time to explode, making it unusable for real-time applications.

The budget impact is equally significant: most LLM providers charge per token used, and chain-of-thought inevitably generates many hidden tokens that are still billed to the user, and the consumption is not trivial.

Enabling reasoning on the most advanced models radically changes the economics of your requests.

Paradoxically, using chain-of-thought on simple problems can actually degrade model performance, causing a kind of cognitive wandering. This finding forces us to develop a strategy where the system must learn, through specific models, to calibrate its reasoning effort.

In practice, delegating basic chat or simple keyword extraction to a reasoning model is like renting a scientific supercomputer to use as a desktop calculator. It's an architectural absurdity. It's imperative to reserve reasoning models for specific cases: algorithmic design, navigating logical mazes, critical evaluation, or the initial planning phase of a complex agentic workflow. Defined, repetitive, execution-only tasks must remain the exclusive territory of fast models.

## The Semantic Traffic Controller

The juxtaposition of these diametrically opposed models in terms of cost and capability introduces a new challenge: how does the system know, before even executing the task, which type of model to route the user's request to? You could use an LLM to evaluate the difficulty level of each incoming request, but that would eliminate all the economic benefits you were trying to achieve.

The solution that has gradually established itself is semantic routing. Positioned upstream in the system, the semantic router acts as an ultra-fast, ultra-cheap traffic controller.

Unlike the LLM-based prompting approach, the semantic router relies on vector space manipulation (embeddings).

The text query is immediately converted into a mathematical vector by an extremely lightweight encoding model such as [ModernBERT](https://huggingface.co/blog/modernbert), which runs locally with minimal memory footprint via optimized libraries like [Rust Candle](https://github.com/huggingface/candle). The system doesn't try to understand the cognitive meaning of the sentence; it mathematically positions that vector in a multidimensional decision space, often held in memory or via dedicated vector databases ([Pinecone](https://docs.pinecone.io/), [Qdrant](https://qdrant.tech/)).

Based on the vector distance between the input text and the routing vector space, the request is routed either to a fast, economical model or to a reasoning model.

Adopting this architecture radically reduces both latency and financial cost.

Moreover, this mandatory chokepoint acts as a cognitive firewall. Before even reaching the inference engines, fast classifier models scan the query vector for prompt injection attempts or toxic content, rejecting malicious requests with near-zero latency and preserving system security.

Only in the absence of any strict match in the vector space, when faced with a completely novel or ambiguous request, does the semantic router fall back and gracefully delegate the decision to a general-purpose LLM.

## Multi-Agent Architecture

RAG for knowledge, function calling for action, complex reasoning, and semantic routing for load distribution: we now have all the fundamental building blocks. Assembling these elements represents the logical culmination of AI software evolution, marking the era of distributed systems in 2026: multi-agent architecture.

Multi-agent architecture relies on an orchestrator delegating tasks to specialized, independent AI processes.

This decentralized approach radically reduces operating costs and increases overall system resilience, since each component is isolated, testable, and replaceable without compromising the software base. Each agent nevertheless remains a probabilistic engine: specialization reduces the error space but doesn't eliminate it.

The ultimate optimization of multi-agent architecture in 2026 comes through the physical redistribution of compute. Instead of concentrating all work on SaaS model API providers, simpler tasks are offloaded to SLMs (Small Language Models), highly optimized neural networks typically under 8 billion parameters, capable of running locally without a network connection.

The SLM becomes a persistent contextual expert. Only when faced with a request outside its competency domain, or requiring greater analytical power, does the supervising agent trigger routing to the SaaS model provider.

## When Models Self-Correct

One of the biggest challenges of enterprise AI is trusting its outputs: how do you know when an AI starts hallucinating or becoming incoherent?

The solution lies in a two-headed architecture:

- A fast, agile execution model that generates content, code, or responses.
- A high-reasoning model that analyzes the first model's work.

This is the "LLM-as-a-judge" concept. Instead of waiting for a human to review every response, you use the superior logic of a reasoning model to score and validate the output of the standard model.

The practical impact: in regulated environments, using an "analytical" judge (such as GPT-4o or Claude 3.5 Sonnet) to evaluate a smaller model drastically reduces false positive rates. Studies on the [MT-Bench benchmark](https://arxiv.org/abs/2306.05685) show that advanced reasoning models achieve over 80% correlation with human judgments, significantly outperforming classical keyword-based verification systems.

This automated control loop is the final safety lock: it certifies that a response is reliable before it ever reaches the user's eyes.

## Outlook: AI as a Resilient System

In just two years, our relationship with language models has radically changed. We've stopped seeing AI as a monolithic, mysterious entity to be "tamed" through magic formulas (the famous "prompt engineering"). Today, the model is just one puzzle piece, a compute engine embedded within much larger hybrid architectures.

This evolution grew from a realistic assessment. We understood that giving AI infinite contexts was pointless if it got lost in them. That's why RAG established itself for navigating massive knowledge bases by filtering out useless noise, which in turn rehabilitated extended context, which remains very valuable for exhaustive analysis of large documents that RAG's chunking sometimes struggles to match.

In parallel, a hierarchy has emerged. On one side, "reflex" models, fast and cheap, whose responses remain probabilistic; on the other, "deep reasoning" models, powerful but slow and expensive. To avoid budget explosions, we've had to build real control towers: intelligent routers that decide, in milliseconds, which model is best suited to answer each question.

In 2026, building an AI solution is therefore a matter of pure systems engineering. The developer no longer just "converses" with a neural network hoping for the right answer. They've become the architect of a software factory where dozens of micro-agents collaborate, monitor each other, and self-correct.

In the end, the real advance may not lie in the power of the models themselves, but in our capacity to organize them. We're no longer building oracles; we're building resilient systems capable of acting on the real world. The question is no longer whether AI can think, but how well we can orchestrate its autonomy.
