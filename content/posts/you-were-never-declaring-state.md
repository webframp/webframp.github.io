---
title: "You Were Never Declaring State. You Were Observing By Hand."
date: 2026-05-22T10:00:00-04:00
tags: [swamp, infrastructure, devops, ai, promise-theory]
description: "What changes when your agent can observe reality instead of reading your notes about it"
---

Every Terraform file you ever wrote was a note to a blind tool. You looked at
your infrastructure, decided what it should be, wrote that decision into HCL, and
handed it to a program that could not see for itself. The declaration was your
observation, pre-computed and frozen into a file.

The same was true for Chef recipes, Puppet manifests, and CFEngine promises. You
observed the system. You wrote down what you saw and what you wanted. You gave
that note to an agent that could parse files but could not look around.

That was never "declaring state." That was observing by hand and writing your
observations into a format a blind agent could consume.

## What the declaration actually was

Mark Burgess described promise theory in
[In Search of Certainty](https://www.markburgess.org/certainty.html) as a model
of autonomous agents reasoning about their environment and making local promises
about their own behavior. The agent observes. The agent decides. The agent acts.

The tools we built to approximate this vision could not do the first step. A
CFEngine agent could parse a promise body and converge toward it, but it could
not observe the broader system and form a judgment about what promises to make.
A Chef agent could execute resources in order and check whether each one needed
convergence, but it could not look across the node's full state and decide
whether the recipe was even relevant.

So we wrote the observation for them. We called it "desired state" and stored it
in files. The file was scaffolding for an agent that lacked the capacity to
observe.

## What changed

Agents can observe now.

A swamp model method runs against a live system, captures what it finds, and
stores the result as typed, versioned, schema-validated data. No human
pre-computed the observation into a file.

The [@webframp/aws/adopt](https://github.com/webframp/swamp-extensions/tree/main/aws)
extension exists solely to do this: observe existing AWS resources and bring them
under management as typed data. It does not declare what the resources should be.
It captures what they are. Each execution produces a versioned snapshot of
reality.

```bash
swamp model method run my-account discover_all --json
```

After this runs, the agent knows what exists: VPCs, subnets, gateways, route
tables, security groups, RDS clusters, secrets. It can query that knowledge:

```bash
swamp data query 'modelName == "my-account" && isLatest == true' --json
```

The data evolves with the live system. Run the method again next week and you get
a new version. Compare versions and you see drift. No declaration file needed
because the agent observed reality directly.

## Where the boundary moved

For a decade, the architect's job included crafting declarations. You decided how
to represent your infrastructure in HCL or Ruby or YAML. The shape of the
declaration was a design decision. You spent judgment on it.

That boundary moved.

The architect's job is now deciding where observation ends and action begins.
Which data is sufficient context for a decision? What workflow conditions justify
a write operation? What model boundary separates awareness from control?

These are boundary decisions, the places where you choose what one system knows
about another. You still spend judgment. You still define shape. But
now you think about the data you want to store more than the syntax of the
declaration you used to write. What schema does the observation produce? What
fields become queryable? What versioning granularity makes drift visible? The
design work moved from HCL blocks to data shape.

## The honest limits

This does not eliminate declarations. Three cases still require stated intent:

**Provisioning.** You cannot observe a resource that does not exist yet. Creating
something new requires you to specify what you want before reality contains it.
This is a small fraction of infrastructure operations work. Most of the work is
managing what already exists.

**Compliance baselines.** "All S3 buckets must have encryption enabled" is a
statement of intent, not an observation of reality. A compliance baseline is a
declaration by definition. But it lives in a Zod schema or a workflow condition,
not in a per-resource YAML block.

**Rollback targets.** "Roll back to the state from Tuesday" requires knowing what
Tuesday's state was. Versioned data provides this (retrieve version N), but
someone must decide which version is the rollback target. That decision is
intent.

In all three cases, the intent lives in method logic, workflow conditions, and
schema definitions. Not in static files that drift from reality between applies.
The declaration moved from a file the tool reads to a constraint the agent
reasons about. Still declarations. Different medium.

## Idempotency moves up the stack

The old tools required every resource to be individually idempotent. A Chef
resource had to check whether the file existed before writing it. A Terraform
resource had to detect whether the security group already had the rule. Every
atomic operation carried its own convergence logic because the agent could not
reason about the broader context.

When the agent can query versioned state before executing, idempotency becomes a
property of the workflow's judgment rather than each substep's implementation. The
agent asks: "has this work already been done?" It checks the latest snapshot. If
the answer is yes, it skips the operation. The individual method does not need
to be idempotent because the workflow decided not to call it.

This connects to a pattern already visible in practice: workflows that check
shared state and decide "nothing changed, skip this" before executing. The
pipeline assumed every stage must run because no stage could reason about whether
it should. The agent checks first.

## What this means in practice

The [@webframp/aws/adopt](https://github.com/webframp/swamp-extensions/tree/main/aws)
workflow produces value from pure observation that compounds across use cases:

**Queryability.** You can run CEL queries across your entire AWS account state.
"Show me all resources tagged production that were created in the last 30 days."
You cannot answer that from a Terraform state file unless every resource is
already under Terraform management.

**Composition.** Other swamp models and workflows consume adopted resource data as
input. "Which security groups allow ingress from 0.0.0.0/0 on port 22?" A
security scanning workflow answers that by reading the adopt output directly. A
compliance workflow reads it to know what baseline to verify against.

**Drift awareness.** Run adopt periodically. Each run produces a new version.
"What changed in my VPC configuration since last Thursday?" Compare versions to
answer that. You compare reality at time T to reality at time T-1, with no
declaration to compare against.

## The principles survive

Promise theory described autonomous agents observing their environment and
reasoning about what to do. We built approximations constrained by the agents
available: daemons that could parse files but not observe. The declaration was the
scaffolding those limited agents required.

The scaffolding is optional now for everything except provisioning, compliance,
and rollback. The principles (convergence, autonomy, local reasoning) are better
served than they were by any generation of static declarations. Burgess was right
about what agents should do. The agents just needed thirty years to catch up.

You were never declaring state. You were doing the observation work that your
tools could not do for themselves.

The tools caught up. The question left for the architect: where do I draw the
line between what the agent observes and what the agent changes?
