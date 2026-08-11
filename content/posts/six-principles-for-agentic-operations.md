---
title: "Principles for Agentic Operations"
date: 2026-08-11T10:00:00-04:00
tags: [ai, devops, swamp, manifesto, promise-theory, deming]
description: "A manifesto"
---

Short enough to remember, precise enough to disagree with.

## 1. Observation is the primary operation

Action follows observation; it never precedes it. A system that acts without
first observing is blind, regardless of how sophisticated its declaration
layer is.

## 2. Scaffolding is removable. Principles are not

Autonomy, locality of knowledge, voluntary commitment, evidence over
authority. These survive every tool generation. Sprint ceremonies, state
files, approval queues, and required reviewers are scaffolding. The work is
knowing which is which.

## 3. Governance flows from observation, not from control

Controls that address observed failure modes are engineering. Controls that
address imagined failure modes are ceremony. For each control in your system,
name the specific failure it prevents. If you cannot, you have added an
inspector.

## 4. Human judgment belongs in system design, not in inspection

The highest-value human work is defining what to observe, what to count
as drift, where to draw trust boundaries, and what evidence to demand.
Reviewing outputs in a queue is maintenance. Designing the system that
produces trustworthy outputs is architecture.

## 5. Precise thinking is the scarce resource

The bottleneck is never the tooling. It is the distance between what someone
vaguely intends and what they can state precisely enough for a deterministic
system to act on. Typed schemas, explicit dependencies, and structured
feedback loops close this gap. They force the precision every other
improvement depends on.

## 6. The substrate before the removal

Four properties must exist before scaffolding is safely removable: trust
infrastructure (failure is safe to report, authority is bounded and
explicit), observable state (typed, versioned, queryable), decision traces
(every judgment leaves a record), and explicit trust boundaries (who
authorizes what, under what conditions). Remove ceremonies without these and
you get chaos with faster iteration.

## What this is for

None of these six are original. Burgess supplied the first two. Deming
supplied the third and fourth. The last two came from twenty-five years
building systems at scale: physical racks, virtualized clusters, and
cloud-native services, each with its own failure modes. Most recently that
means crafting [swamp](https://swamp-club.com/)
[extensions](https://github.com/webframp/swamp-extensions/), load-testing
the datastore beneath them until it buckled, and reading the reports
afterward.

Only the occasion is new: agents that can observe a live system, reason
about what they find, and act on that reasoning without a human translating
the observation into a file first.

That capability is roughly two years old, and the theory for operating
systems built on it does not exist yet. Chef, Puppet, Terraform, and Ansible
took thirty years to reveal how little of promise theory they actually
implemented, because thirty years was what it took to accumulate enough
failures to see the pattern. Nobody has run agentic operations long enough
to have that catalog yet, and nobody should need thirty years to build one.

Take them, break them, cite the case that breaks them. That is the only way
a manifesto earns the word, and the fastest way to find out how agentic
operations actually fails.
