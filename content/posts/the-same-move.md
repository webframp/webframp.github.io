---
title: "The Same Move"
date: 2026-06-09T10:00:00-04:00
tags: [devops, ai, swamp, workflow, deming]
description: "Declarations were scaffolding for blind agents. Ceremonies were scaffolding for blind organizations. The same move removes both."
---

The declaration authorized the reconciler and the ceremony authorized the team,
and neither required a trust decision because the scaffolding decided for them.

The declaration told the blind tool what reality should look like. The ceremony
told the blind organization what the work should look like. Both existed because
the system could not observe for itself, so we built structures that made
observation unnecessary by embedding the answer in advance.

Those structures are now removable. The question worth asking is what has to
exist before you can safely remove them.

## The blindness they compensated for

A Terraform file was your observation of infrastructure, pre-computed and frozen
into HCL. You looked at the system, decided what it should be, and handed that
decision to a tool that could not look for itself. The tool converged toward your
description of reality rather than toward reality itself.

A sprint planning meeting was your observation of work, pre-computed and frozen
into a backlog. You looked at what needed doing, decided what the team should
commit to, and handed that decision to a process that could not see the work
flowing through the system. The process executed your description of capacity
rather than observing capacity itself.

In both cases the scaffolding solved a real problem. Agents that cannot observe
need declarations. Organizations that cannot see the work need ceremonies. The
scaffolding was not a mistake. It was a correct response to a genuine constraint.

The constraint has changed.

## What observation looks like now

On the infrastructure side, agents observe directly. A swamp model method runs
against a live system, captures what it finds, and stores the result as typed,
versioned, schema-validated data. No human pre-computed the observation into a
file. The agent compares observed state at T1 with observed state at T0, and that
comparison is drift detection without any declaration as an intermediary.

On the organizational side, the work is observable by default. Typed state,
decision traces, continuous delivery, agent execution logs. You do not need to
gather people in a room and ask them to narrate what they did yesterday. The
system shows you. The daily standup was solving a visibility problem that
well-instrumented systems already solved.

Once the system can see, the workaround becomes overhead.

## What fills the gap

Removing scaffolding without building a replacement produces chaos at both
layers. The declaration was load-bearing: it told the reconciler what to do
without requiring a trust decision at runtime. The ceremony was load-bearing: it
gave the team permission to proceed without requiring anyone to observe the
system continuously.

If you strip both and put nothing in their place, you get infrastructure that
observes drift endlessly without acting and teams that surface signals without
deciding. The gap between observation and action is where things break.

What fills it: a reasoning agent, human or trained LLM, constrained by typed
context, operating within explicit trust boundaries, with decision traces that
make every judgment auditable.

For infrastructure, this means separating observation from action deliberately.
Observation runs continuously, automatically, cheaply. Action is episodic,
authorized, expensive. The agent presents the diff between observed states and
waits. A human or a trained agent with explicit scope decides whether to act. The
Kubernetes operator embeds intent in the control loop and converges
automatically. The observation model puts a reasoning agent in the gap between
seeing and doing.

For teams, this means replacing ceremonies with continuous signal surfaces and
explicit coordination policies. A two-week sprint batches commitments that WIP
limits and priority ordering already make visible. A daily standup surfaces
blockers that the system already shows you. The coordination function survives.
The meeting that used to proxy for it does not.

In both cases the replacement has the same structure: continuous observation,
explicit trust boundaries, episodic authorized action, and a decision trail.

## What goes wrong without the substrate

Build continuous observation without building the authorization layer and you get
systems that detect every drift and act on all of it simultaneously. Two hundred
reconciliation loops firing at once because a node went down and every controller
watching those pods decided to act in the same moment. The old declaration
prevented this by limiting scope: the reconciler only touched what the
declaration described. Without declarations, the scope constraint has to live
somewhere else, in the trust boundary of the reasoning agent.

Strip ceremonies without building generative culture and you get teams that move
fast in random directions. Westrum called these generative organizations: they
focus on the mission, encourage collaboration, and make failure safe to report.
Pathological and bureaucratic organizations use the same tools to automate their
dysfunction faster. Stripping ceremonies in a bureaucratic organization produces
chaos with faster iteration and no coordination.

There is a subtler failure at both layers: human blindness to what the system
decided not to do. An observation loop that detects no drift and takes no action
leaves no trace of its judgment unless you build the trace in. A team with no
ceremony and no signal surface has no visibility into what was considered and
rejected. The absence of action is invisible by default. Making it visible
requires the same move in both domains: write the observation even when nothing
changed, because the absence of drift is the finding.

## The substrate

In the Blue Mountains of Eastern Oregon, a single organism covers 3.5 square
miles beneath the Malheur National Forest. *Armillaria*, the honey mushroom,
spreads as black fibers through interconnected tree roots, traveling from tree to
tree as a thin white layer under bark. Most of the year it is invisible to
anyone walking above it. The mushrooms that occasionally fruit on the surface are
indicators, not the organism. The organism is the network underneath.

The forest does not need the fruiting bodies. It needs the mycelium.

Declarations and ceremonies were fruiting bodies. Visible, recognizable,
apparently important. But the organism that actually held things together was
always the substrate: trust, shared context, observable state, traceable
decisions. When the substrate is healthy, the surface structures are optional
indicators. Remove them and the system continues because the connections run
deeper. Remove the substrate instead and nothing connects, regardless of how many
ceremonies you schedule or declarations you write.

What has to exist before the scaffolding is safely removable:

**Trust infrastructure.** People who trust each other enough to surface problems
immediately rather than saving them for a scheduled meeting. Agents with explicit,
bounded authority rather than unbounded convergence loops. Trust here means a
structural property of the system: does it make failure safe to report, and does
the agent's scope match the operator's actual delegation?

**Observable state.** Typed, versioned, queryable data at both layers. Not logs
that someone might grep. Not dashboards that someone might check. State that
agents and humans can query programmatically, compare across time, and reason
about as structured input.

**Decision traces.** Every judgment, whether by a human in a design session or an
agent evaluating a diff, leaves a record of what was considered, what was
decided, and why. The ceremony used to produce this as a side effect: sprint
planning created a backlog, the daily standup surfaced context. Without
ceremonies, the trace has to be built deliberately into the observation and
authorization system.

**Explicit trust boundaries.** Who can authorize what, under what conditions,
with what scope. The declaration was an implicit trust boundary: the reconciler
could only touch what the HCL described. The sprint was an implicit trust
boundary: the team only worked what they committed to. Both need explicit
replacements.

These four properties are the same at both layers because the problem is the
same at both layers: you are removing a structure that made authorization
implicit, and you need to make authorization explicit without reintroducing the
overhead the structure created.

## The principles survive

The agile manifesto was right about what teams should value: working software,
responsiveness to change, people over process. Scrum
was scaffolding for organizations that could not yet operate on those values
without explicit coordination structure. The ceremonies were training wheels.

Burgess was right about what agents should do: observe, reason locally, make
promises about their own behavior. Chef and Terraform were scaffolding for agents
that could not yet observe. The declarations were training wheels.

In both cases the principles were always the destination. The scaffolding was the
cost of getting there with the systems available at the time. What changed is not
the principles. What changed is the capacity of the systems to embody them
directly, without the intermediary structures we built to compensate for their
limitations.

The hard question for organizations right now: can you tell the difference
between your scaffolding and your principles? If you think Scrum is agility, you
will fight to keep ceremonies that have become pure overhead. If you think the
Terraform file is your infrastructure, you will fight to keep declarations that
have become pure overhead.

The scaffolding is removable. The principles are not. The work is knowing which
is which.
