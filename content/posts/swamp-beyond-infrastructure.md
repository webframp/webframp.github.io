---
title: "Swamp Beyond Infrastructure"
date: 2026-05-18T11:01:17-04:00
tags: [swamp, ai, writing, workflow]
description: "The design space that opens when you stop thinking in domains"
---

If you have used [swamp.club](https://swamp.club) at all, you probably think
of it as an infrastructure automation tool. The ecosystem gravitates that way.
The most-pulled extension is @john/k8s with 15 model types wrapping the
Kubernetes API. The examples in the docs show pod health checks and deployment
workflows. The leaderboard is full of people automating infrastructure tasks.

That framing is incomplete. What swamp actually provides is a typed, versioned,
schema-validated data layer for AI agents. The primitives are: models with [Zod](https://zod.dev/) 
schemas, immutable versioned data, method execution, and composable workflow
DAGs. Those primitives do not care whether the data flowing through them is pod
status or prose rules.

I discovered this sideways, looking at an extension called
[@dougschaefer/writing-voice](https://swamp.club). It stores an organization's
entire editorial voice as structured data so an AI agent can load it before
generating content and sound like the organization instead of defaulting to
generic AI prose.

It uses the exact same architecture as the k8s extension. Same model/method/data
contract. Same versioning. Same schema validation. The tool does not know the
difference.

## What the extension does

One TypeScript model. Two methods. No network calls. Sub-25ms execution.

The `get` method serializes a voice profile into versioned data. The
`add-reference` method stores annotated writing examples. That is the entire
surface area.

The voice profile has eight sections:

- **Identity**: who the organization is when it writes
- **Tone tiers**: full register, polished, and formal variants
- **Prose rules**: mechanical rules like "one thought per sentence"
- **Positioning framework**: the author persona
- **Document types**: blog post, ADR, commit message, each with structure
- **Audience calibrations**: what engineers need vs. what leaders need
- **Anti-patterns**: wrong/right pairs with explanations
- **Kill list**: phrases that never appear

The anti-patterns section is the most effective part. Abstract rules like "do not
hedge" produce inconsistent results from an LLM. Concrete wrong/right pairs
produce measurably better output because the model can pattern-match against
specific failures:

```yaml
antiPatterns:
  - wrong: "This solution helps to potentially address some of the challenges..."
    right: "This fixes the timeout bug in the connection pool."
    explanation: "State what it does. Do not hedge about what it might do."
```

A companion skill loads this data before content generation:

```bash
swamp data get editorial-voice voice-profile --json 2>&1 | jq -s '.[0].content'
```

The agent gets typed, validated, versioned context. Not a blob of prior
conversation. Not a system prompt that drifts every time someone edits a Google
Doc.

## The same pattern everywhere

This is not a one-off. Look across the 663 extensions in the ecosystem:

| Extension | Domain | Pattern |
|-----------|--------|---------|
| @john/k8s | Infrastructure | kubectl output → structured data → workflows compose across resource types |
| @mellens/rave | Reliability engineering | Claims with confidence scores, evidence from GitHub/Prometheus/CI, exponential decay |
| @magistr/good-planning | Governance | State machine with `hydrate()` for agent-friendly summaries, Zod-first output schemas |
| @dougschaefer/writing-voice | Content | Voice profile → versioned data → agent loads before generating |
| @bixu/wheelshop | Dependency vetting | npm/jsr search with 8 trust gates, guardrails for extension authors |
| @dougschaefer/utelogy | AV monitoring | Conference room device metrics and CLM status |
| @keeb/prometheus | Observability | SSH-based agent deploy for monitoring stacks |

Every one follows the same contract: define a model, define methods that produce
typed output, store the output as versioned data, compose methods into workflow
DAGs. Whether the data is a Kubernetes deployment object or an editorial kill
list, the machinery is identical.

Compare the k8s extension producing structured pod data:

```bash
swamp model method run k8s-pods get --json
```

Against the writing-voice extension producing a voice profile:

```bash
swamp model method run editorial-voice get --json
```

Same command shape. Same schema validation. Same versioned output. Queried the
same way with CEL. The domain vanishes at the contract boundary.

## Why this matters for agents

As [Paul Stack](https://stack72.dev/) put it, swamp brings determinism to a
probabilistic system. Four things happen when your writing voice lives in this
kind of architecture.

**Agents get structured context, not chat history.** Loading a voice profile from
`swamp data get` before writing is the same pattern as loading cluster state
before diagnosing. The agent receives a typed object with specific fields it can
reason about. Not "here is a long conversation, figure out what matters."

**Voice evolution becomes trackable.** Writing voice drifts. With five retained
versions, you see how your positioning changed. You can roll back when a revision
does not work. This is version control for editorial decisions, the kind that
usually lives in someone's head or in the revision history of a Google Doc that
nobody reads.

**Voice becomes deployable infrastructure.** The extension is published.
`swamp extension pull @dougschaefer/writing-voice` and you have it. An
organization's voice becomes an artifact you install and configure, not tribal
knowledge. New team members load the profile and get concrete wrong/right pairs
instead of a style guide PDF.

**Workflows can compose voice with other concerns.** Load voice profile → load
audience calibration → generate content → check against anti-patterns → score
output. Each step is a typed model method. The workflow DAG handles
orchestration. This is the same composition model that strings together health
checks across five services in an infrastructure workflow.

## The thesis

Swamp crossed one million automation events in early May 2026. Most of those
events are infrastructure. But the reason the tool works for writing voice is not
that someone hacked a creative use case onto an infrastructure platform. It is
that the primitives were right from the start.

Schema in. Validated data out. Versioned and queryable. Composable into DAGs.

Those four properties do not privilege any domain. A writing voice profile is
just data with a schema. An anti-pattern is just a typed object. A kill list is
just an array of strings with a validation contract. The moment you stop thinking
of swamp as "the k8s workflow tool" and start thinking of it as "typed versioned
data for agents," the design space opens up.

The infrastructure community found swamp first because they had the most
immediate pain: agents running kubectl and parsing stdout. But the same pain
exists everywhere an agent needs structured context to do its job. Content
generation. Reliability assessment. Governance tracking. Dependency vetting.
Conference room monitoring.

Every AI writes like it was trained on corporate annual reports. The voice
profile stored in swamp is what makes the difference between technically correct
and completely forgettable versus prose that sounds like actual people who have
done the work. And it is stored in the same system that manages your pods.

That is not a coincidence. That is what good primitives get you.

What else in your routine is unstructured context that an agent needs before
it can do good work? Your incident response playbooks? Your hiring rubrics? Your
product positioning? If it can be expressed as a schema, it can live here.
