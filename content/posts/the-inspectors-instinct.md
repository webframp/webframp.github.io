---
title: "The Inspector's Instinct"
date: 2026-06-03T10:00:00-04:00
tags: [devops, ai, swamp, deming, governance]
description: "The governance you know is the governance you repeat."
---

You have agents running in your system. Your first instinct was to add controls.
Approval gates before they execute. Escalation policies when they're uncertain.
Manifests declaring what they're allowed to do. Kill switches for when they go
wrong.

This is the same instinct that produced the approval queue for humans. You are
governing agents the way you governed people, because that is the only governance
you know.

Deming had a name for the knowledge required to manage a system well. He called
it the System of Profound Knowledge.[^1] Four components, never meant as a
checklist: what is the system, where does it vary, how do you know what you know,
and why do people behave the way they do.

You don't need to know the framework to benefit from it. You need to ask the
questions it asks.

## The system you are not seeing

The "agentic mesh" conversation in the industry right now makes a useful
observation: value emerges from the connections between agents, not from
individual agent capability. The proposed solutions are domain ownership,
agent-as-product contracts, governance pillars, registries, manifests. Declare
the system's shape upfront. Govern through contracts.

The instinct is understandable. But declaring a system's shape is not the same as observing it.

In [Swamp](https://swamp.club), the system's shape is visible in the data layer.
Typed models, versioned artifacts, DAG dependencies. You don't need a manifest to
declare what an agent can do if every execution writes what it did. The system
becomes observable through its own operation.

Do you understand the system as a system? Not as a collection of parts, but as
the interactions between them. The typed data layer is where those interactions
become visible. When agent A's
output feeds agent B's input through a shared datastore, the relationship is not
declared in a JSON file. It exists in the versioned data. You can query it. You
can trace it. You can see when it breaks.

The practitioners who build agent systems without this visibility are flying
blind. They add governance scaffolding because they cannot see the system's
actual behavior. The scaffolding is a substitute for observation.

## Variation is information

Agents produce variable outputs. Same prompt, same context, different result.
Most teams respond by constraining the agent further (tighter prompts, smaller
decision scope, more guardrails) or by adding inspection after the fact (human
review of every agent output).

Both responses misunderstand what variation means in a system.

Variation is information about how the system behaves. When your agent produces different outputs across runs, the question is
whether that variation is signal or noise. You can only answer that question if
you are capturing the data.

In swamp, every method execution writes versioned, typed data. Run the same
method ten times and you have ten snapshots to compare. CEL queries across those
snapshots reveal where variation concentrates. Is it in the external system's
responses? In the agent's interpretation? In the model's reasoning? The data
layer makes variation visible without adding an inspector to judge each
individual output.

The contract-first approach (agent cards, SLOs, decision-quality metrics) tries
to solve this same problem by defining acceptable variation upfront. That works
when you already know what acceptable looks like. It fails during the period when
you are still learning what the system does. Observation-first design lets you
learn the system's variation profile from actual behavior, then write constraints
from evidence rather than guessing them in advance.

## What the agent knew when it decided

How does the system know what it knows? In most agent setups, knowledge lives in
prompts, context windows, and RAG retrievals. None of it is versioned or
queryable after the fact. When an agent makes a decision, you cannot
reconstruct what it knew at the time it decided.

Swamp's typed data layer makes this concrete. The compliance baseline the agent
evaluated against is versioned. The vulnerability data it consumed carries a
timestamp. And the workflow DAG records which inputs produced which outputs.
When something goes wrong, you can reconstruct the agent's state of knowledge at
the moment of decision. Not "what was in the prompt" but "what typed data existed
in the datastore at execution time, and what query produced the input the agent
acted on."

This is where the registry-and-manifest approach genuinely contributes something.
Discoverability across domains is a real problem. If agent A in one team needs to
consume the output of agent B in another team, how does it find that output?
Swamp's answer today is shared datastores. That works within a team.
Cross-domain discovery remains an open problem, one where registries add value
even if the registry should describe observed capabilities rather than declared
intentions. Maybe the giga-swamp becomes the answer here.

The honest gap: swamp can tell you what an agent knew. It cannot yet tell you
what an agent in another organization's swamp instance knew. The mesh
conversation is trying to solve coordination at scale. The disagreement is about
method (declare upfront vs. observe and publish), not about whether the problem
matters.

## The governance instinct

Why do people add governance scaffolding to agent systems? Not because the system
needs it. Because the people around the system need to feel safe.

The kill switch. The escalation ladder. The approval gate before production.
These exist to manage anxiety, not risk. The risk was managed, or not, by the
system design. The ceremony manages the humans' relationship to uncertainty.

Deming's fourth question was about psychology: you cannot improve a system
without understanding why the people in it behave the way they do. Teams add
inspectors to agent systems because that is how they managed uncertainty about
human workers. The instinct is organizational muscle memory. It is not wrong to
feel it. It is wrong to encode it into infrastructure without asking whether it
addresses a real failure mode or merely a familiar fear.

Here is the practitioner's question, the one worth taking back to your desk:

For each control you have added to your agent system, can you name the specific
failure it prevents? Not a category of failure. A specific scenario, observed or
reasoned from the system's actual behavior. If you cannot, you have added an
inspector. You are managing your psychology, not your system.

This is not an argument against safety. It is an argument for grounded safety.
Controls that address observed failure modes are engineering. Controls that
address imagined failure modes are ceremony. The difference between the two is
whether you have looked at the data.

## The slow work

Designing systems that are trustworthy by construction requires understanding
them deeply enough to know where trust breaks. That understanding takes time,
observation, and willingness to let the system show you its failure modes rather
than guessing them in advance.

Most teams do not have that patience. They add gates because gates are fast to
build and legible to leadership. A kill switch is visible. A well-designed typed
data layer is not. The slow work of system understanding is invisible, and invisible
work is the first thing cut when someone asks for a status update.

Deming knew this. He spent forty years arguing that the gap between knowing a
system and controlling a system is where most organizations live. Agents do not
close that gap. They only make it more expensive to ignore.

[^1]: W. Edwards Deming, *The New Economics for Industry, Government, Education* (MIT Press, 1993). The System of Profound Knowledge is the framework of his later career, synthesizing his earlier work on variation and systems into a unified theory of management.
