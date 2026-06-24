---
title: "The Mirror You Trained"
date: 2026-06-24T10:00:00-04:00
tags: [ai, swamp, thinking, systems]
description: "The LLM does not create the thinking gap. It makes the gap visible and consequential for the first time."
---

Hand someone a typed model in [swamp](https://swamp-club.com/) and ask them
to define what data they want to capture. Watch what happens. The schema
requires them to name the fields, specify the types, declare what
constitutes a valid observation. Most people stall. Not because the tooling
is hard. Because they have never been asked to articulate what they actually
want to know with that level of precision.

The same pattern repeats at every layer. Ask them to write a workflow DAG
and they cannot specify which steps depend on which, because they have never
separated their assumptions about sequence from actual causal dependencies.
A CEL query against versioned data exposes the same gap from a different
angle: you must know the question before you can ask it, and most people
have never distinguished "what do I want to know" from "what does the
system happen to show me."

The tooling is surfacing a thinking problem.

I have watched this enough times to recognize the shape. The difficulty is
never the syntax. It is always the same gap: the distance between what
someone vaguely intends and what they can state precisely enough for a
deterministic system to act on. A typed schema does not accept vague intent.
It requires you to know what you mean.

An LLM has the same property, but without the structured feedback. The
difference in output quality tracks directly to the difference in input
precision. Precise constraints and explicit success criteria produce output
you can evaluate. Vague intent produces output nobody can evaluate, because
there was never a stated standard to evaluate against.

I owned an African Grey parrot for years. The bird was brilliant in ways
that surprised people who had never lived with one. It learned my moods. It
offered me objects at specific times that correlated with my emotional
state. It echoed back phrases it had learned from me, and deployed them in
exactly the right context to communicate what it needed. The phrases were
mine. The deployment was the bird's intelligence. But the raw material was
always what I had given it, and the quality of what came back depended
entirely on the quality of what went in.

If AI is the stochastic parrot that its critics claim, then the mirror it
holds up reflects the thinking you feed it. When the output is generic and
forgettable, that is a diagnostic of the input. Vague input, vague output.
The mirror does not lie, and it does not flatter.

A programmer posted on Substack recently about the death of taste. His
coworkers ship marketing copy through ChatGPT, the results are
indistinguishable from what they wrote before, nobody notices, and this
terrifies him. He watches people accept whatever comes back without
evaluating it against any stated intent.

He is diagnosing a real symptom. He is mis-attributing the cause.

The taste collapse he observes did not begin when the LLM arrived. It began
when organizations spent fifteen years optimizing internal communication for
skimmability. For TL;DR headers. For Slack-message attention spans. For
quarterly review decks that nobody reads past slide three. The LLM walked
into a room full of people who had already been trained to produce and
consume language at the shallowest possible depth. It handed them a tool
that operates at exactly that depth, faster.

This is a systems observation, not a judgment about individuals. Deming
spent forty years pointing at the same structure: when the system produces a
consistent output, the system is the cause. The programmer on Substack sees
his colleagues producing mediocre output with AI assistance and concludes
they lack taste. What he is actually observing is an environment that never
selected for taste in the first place.

A typical knowledge worker moves through systems that never ask for
precision. Slack optimizes for speed, not depth. Streaming platforms
redesign their content for what the industry calls the "second screen"
problem: shows rewritten with simpler dialog and obvious cues so distracted
viewers will not miss the plot. Social media rewards the fastest emotional
response, not the most considered one. None of these systems train the skill
that good AI interaction requires: the ability to know precisely what you
think and articulate it clearly enough for another entity without your
context to act on it.

People are capable of far more precise thinking than their environment has
ever required of them. The gap between capacity and routine practice is
enormous, and it persists because every system in a knowledge worker's week
optimizes for throughput, not depth.

The LLM does not create the thinking gap. It makes the gap visible and
consequential for the first time.

One response is to ban the tool, which addresses the symptom by removing
the mirror. Another is to blame the user, which locates a systemic failure
in individual character. Both miss what is actually available: build systems
that close the gap.

The thinking that produces good AI output is the same thinking that produces
good engineering, good writing, good decision-making without AI. Clear
intent. Precise articulation. Explicit constraints. Willingness to review
output against intent and iterate when they diverge. These are skills, and
skills respond to training environments.

In swamp, the training environment already exists. A typed model with a Zod
schema forces you to articulate what data you want to capture before you
capture it. A workflow DAG makes dependencies and conditions explicit. A CEL
query against versioned data requires knowing the question before you ask
it. None of these steps involve probability. They are deterministic
structures that require clear human intent as input, and they produce
verifiable, versioned, queryable output.

The agent runs inside those structures. When the model method produces typed
data that does not match what you expected, the schema tells you exactly
where intent and outcome diverged. When the workflow step fails a condition,
the DAG shows you which dependency produced the unexpected state. The system
gives you feedback loops that the raw LLM interaction does not: structured,
typed, versioned evidence of what your thinking produced when it hit
reality. You can query the history. You can compare version three against
version seven and see where your understanding shifted.

Compare this to the feedback vacuum the programmer on Substack is
witnessing. His colleagues generate, ship, and never close the loop. No one
stated intent precisely enough for drift to be measurable. They have a tool
and no training environment for using it well. A
[code review](https://swamp.club/extensions/@webframp/gitlab-review)
workflow built in swamp is the opposite: the agent fetches a diff, produces
a typed review draft, and stores it as versioned data. You revise it. The
revision is a new version. When you look at version one against version four
you can see where your criteria sharpened, where your understanding of what
matters in a diff evolved. The loop closes because the system retains your
thinking across iterations and shows you the trajectory.

A [DDD guidance](https://swamp.club/extensions/@webframp/ddd-guidance)
model does the same for domain understanding. It stores the results of
discovery conversations as typed, versioned resources. Twenty versions of a
context map show how your understanding of your own system deepened over
months. Early versions capture assumptions. Later versions capture what you
learned when those assumptions hit production. The agent asked the
questions. You did the thinking. The data layer kept the record, and the
record trained you.

The feedback and versioning are automatic. The human part is the design:
deciding what to model, what to observe, what conditions constitute
success.

The mirror is already in the room. What matters now is what you build
around it.
