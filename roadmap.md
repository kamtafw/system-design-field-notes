# ROADMAP

## Build my understanding brick by brick

I would arrange my learning into ten stages.

This is a dependency-based roadmap, not a ten-week curriculum. Some stages will take considerably longer than others, and I can revisit earlier concepts when later work exposes a gap.

I've arranged the stages so that I move from understanding system requirements to designing components, reasoning about distributed behaviour, and eventually evaluating an architecture as a whole.

## STAGE 1: Learn to think in systems

**The central question: What are we actually trying to build, and what must it do well?**

This is a starting point for everything else.

Before deciding whether I need a queue, cache, additional database, or separate service, I need to understand the requirements the architecture must satisfy.

### Concepts to investigate

- System boundaries and dependencies
- Functional versus non-functional requirements
- Latency versus throughput
- Availability versus reliability
- Consistency versus durability
- Performance, scalability, and capacity
- Cost and operational complexity
- Architecture diagrams and levels of abstraction

I should learn to distinguish between what a system must do from the qualities it needs to maintain while doing it.

For instance, a functional requirement might be that an inspector can submit a completed field report.

The non-functional requirements might specify that the inspector can complete the report without an internet connection, that submitted reports aren’t silently lost, and that the application eventually synchronizes the report when connectivity returns.

Those requirements also have direct architectural consequences.

### Practical exercise

I will take an application and describe its primary user journeys

For each journey, I will identify:

- Who initiates it?
- Which components participate?
- What data is created or changed?
- Which dependencies must be available?
- What happens if a dependency is unavailable?
- What would constitute an unacceptable experience?

I will then draw a high-level architecture diagram.

Probably, a C4 model for this. It provides different levels of architectural detail, from system context to containers, components, and code. I generally don’t need every level for every system; the context and container diagrams are a useful starting point.

### What I should produce

A short system brief containing the main requirements, constraints, assumptions, and an architecture diagram.

I should be able to explain why the system needs its principal components before explaining how I would implement them.

**I only move on when**: I can describe the system, its boundaries, its essential requirements, and its most important constraints without immediately jumping to a list of technologies.

## STAGE 2: Understand application structure and data ownership

**The central question: How should responsibilities and data be organized inside a system?**

## STAGE 3: Understand requests, performance, and basic scaling

**The central question: What determines whether a system can handle its workload efficiently?**

## STAGE 4: Learn about caching and alternative read strategies

**The central question: How can a system avoid repeatedly doing expensive work?**

## STAGE 5: Understand asynchronous processing and messaging

**The central question: What work needs to happen now, and what work can happen later?**

## STAGE 6: Understanding distributed systems, consistency, and concurrency

**The central question: What happens when multiple components operate independently but must still produce correct results?**

## STAGE 7: Master offline-first and real-time design

**The central question: How do you keep users productive when the client, server, and network don't behave as one continuously connected system?**

## STAGE 8: Learn reliability and operation architecture

**The central question: How do you know the system is working, and what happens when it isn't?**

## STAGE 9: Treat security as an architectural concern

**The central question: What must the system protect, and how do its architectural decisions affect that protection?**

## STAGE 10: Learn how architectures evolve

**The central question: When should the architecture change, and what should trigger change?**