---
title: "Skin in the Game"
date: 2026-05-30
tags: [devops, ai, swamp]
description: "What a development system looks like when you build it to keep yourself honest"
---

I have shipped over thirty extensions for [Swamp](https://swamp.club) in the
last two months. Models for AWS, Cloudflare, GitHub, Reddit, Redmine,
PostgreSQL. Vaults, datastores, workflows, reports. All of it generated
by an AI agent. And nearly all of it would have been structurally wrong without
a system designed to catch structural mistakes before they ship.

The agent writes fast. It writes plausibly. It writes code that passes type
checks and lint on the first try. What it does not do, reliably, is reason about
the system-level consequences of its local decisions. It will generate a
paginator that works for 50 results and silently degrades at 50,000. It will
reuse an instance name across two methods because the name looks right in
isolation. It will swallow a rate-limit header because the happy path never
triggers the branch.

These are not hallucinations. They are architectural blind spots. The code is
locally correct and globally fragile. You only catch them if you understand how
the pieces compose, and you only maintain that understanding by living in the
system every day.

Here is how I stay in it.

## The Loop

Every extension follows the same cycle:

```
plan → generate → adversarial review → fix → iterate → ship
```

The plan is mine. I write it in plain English: what the extension does, what
methods it exposes, what the data shape looks like, where the boundaries are.
The agent generates from that plan. Then the system pushes back.

The system has three layers of constraint, each encoding a different class of
knowledge.

## Layer 1: Accumulated Rules

The project's `CLAUDE.md` contains rules that started as bugs. Every time I find
a structural problem the agent produces repeatedly, it becomes a rule the agent
reads on every invocation.

Two examples:

> **Bounded pagination is mandatory.** Never use `Infinity` or unbounded loops
> for API pagination. Cap fetch limits to a practical multiple and set a
> `truncated: boolean` field in the output when results may be incomplete.

> **Instance names must be collision-resistant.** For variable-length ID lists,
> hash the sorted IDs rather than joining/truncating. Truncated joins produce
> collisions.

These rules exist because the agent produced exactly these mistakes, in
production code, and I caught them after they shipped. The rule is the scar
tissue. It prevents the same class of injury from recurring, across every
extension the agent touches.

The key: rules accumulate from experience, not from speculation. I do not write
rules for problems I have not encountered. Each one traces back to a specific PR
where something went wrong.

## Layer 2: The Adversarial Review

Every PR triggers an automated code review that treats the diff as hostile. It
runs locally before I push (catching problems in seconds) and again in CI on
every push to the PR.

The review prompt encodes specific patterns to verify:

> When you find a pattern (like `.catch()` on batch queries,
> `sanitizeInstanceName()`, truncated tracking, `.max()` on arguments), you MUST
> verify it is applied CONSISTENTLY across ALL methods in the same file that use
> the same pattern. Inconsistency between sibling methods is a HIGH finding.

This is the rule I could not enforce through `CLAUDE.md` alone. The generating
agent applies patterns where it remembers to. The reviewing agent checks whether
the pattern was applied everywhere it should be. Two agents with different jobs
and different failure modes, each compensating for the other's blind spots.

The review produces numbered findings with file, line, breaking scenario, and
fix. Critical findings block merge. I work through them, push fixes, and the
review runs again. The loop repeats until it passes or until I override with
justification.

## Layer 3: The Workflow Skill

A skill called `pr-workflow` encodes the full lifecycle:

```
branch → develop → push → PR → CI + adversarial review → fix → push → repeat → merge
```

The agent cannot skip steps. It cannot merge without CI passing. It cannot
dismiss findings without explanation. The skill is not documentation: it is the
process the agent follows when I say "ship this."

The distinction matters. Documentation describes what you should do. A skill
constrains what the agent will do. One requires discipline. The other enforces
it.

## What the System Catches

Three real examples from the last sixty days:

**Instance-name collisions.** The GitHub extension had two methods, `list_prs`
and `list_issues`, generating artifacts with the name `{owner}-{repo}-{state}`.
Identical names. Data from one method silently overwrote the other through
version stacking. The fix was trivial (prefix with `prs-` or `issues-`), but the
bug was invisible in isolation. You only see it when you understand how Swamp's
data layer resolves names across methods.

**Pagination dishonesty.** Across multiple AWS extensions, the agent generated
paginators that hardcoded `truncated: false` because the code path that fetched
results always completed its loop. But client-side filtering happened after
pagination. If you requested 10 results with a filter, the paginator might fetch
10, filter to 3, and report "not truncated" because it consumed all pages. The
data contract was violated. Three separate extensions had this bug before I wrote
the rule that prevents it.

**Rate-limit fragility.** The Reddit extension's API client parsed rate-limit
headers with `parseFloat` but never checked `isFinite`. A malformed header would
produce `NaN`, which silently disabled rate limiting entirely. The paginator also
failed to break on empty response pages, creating an infinite loop on deleted
subreddits. Neither bug surfaced in normal testing. The adversarial review caught
the `parseFloat` issue; I caught the pagination bug during architectural review
of the client's control flow.

## What the System Cannot Catch

The adversarial review is good at pattern consistency, null safety, injection
vectors, and data integrity. It operates on the diff. It reasons about code.

It cannot tell you whether the extension should exist. It cannot tell you that
your data model conflates two concerns that should be separate. It cannot
evaluate whether a method's boundary is drawn in the right place, or whether the
workflow's job dependencies encode a false constraint.

Those are architectural judgments. They require understanding the domain, the
user's workflow, and the system's trajectory. No review prompt captures that
context. You carry it, or you do not have it.

This is why living in the system is non-negotiable. Not because the tools cannot
handle complexity, but because the judgment that guides them must come from
somewhere. If you stop making architectural decisions daily, you lose the context
that makes those decisions sound. The rules decay. The review prompts miss new
classes of problems. The system drifts toward plausible output with no one
checking whether plausible is correct.

## The Feedback Loop

The system improves because I use it. A new bug class surfaces, and it becomes a
rule. A recurring review finding hardens into a conformance test. A manual step I
keep forgetting becomes a skill that the agent follows automatically.

The number of rules in `CLAUDE.md` is a rough proxy for how much I have learned
about the failure modes of AI-generated code in this specific domain. Thirty
extensions in, the rules cover pagination, naming, null coercion, timestamp
normalization, filtering semantics, and concurrency. Each one prevents an entire
class of bugs across every future extension.

The system is not a replacement for understanding. It is the mechanism by which
understanding compounds. You stay in the game by building the game.
