---
title: "Promise Theory Was Waiting for the Reasoning Agent"
date: 2026-08-07T10:00:00-04:00
tags: [swamp, ai, agents, promise-theory, infrastructure, devops, terraform, ansible, cfengine]
description: "Promise theory described agents that observe, reason, and commit. The IaC generation never delivered it and CFEngine came closest. LLMs supplied the missing reasoning layer, and swamp is built for the agent that has it."
---

Mark Burgess shipped CFEngine in 1993, formalized promise theory with Siri
Fagernes around 2005, and published *In Search of Certainty* in 2015. The model
is precise: autonomous agents observe their environment, reason locally about
what they find, and make promises about their own behavior. The agent does not
receive instructions from a controller. The agent looks around, decides what to
do, and commits to doing it.

Every infrastructure tool since has claimed some lineage from this idea. Chef
called its resources "convergent." Puppet called its catalogs "desired state."
Terraform called its plans "declarative." Ansible called its playbooks
"idempotent."

None of those four kept the promise. Burgess's own tool did, and ran into a
different limit.

The reason to revisit any of this now is that the missing piece finally shipped,
and it arrived from outside infrastructure entirely. LLMs supply the reasoning
layer promise theory always assumed: an agent can observe a live system, weigh
what it finds against context nobody enumerated in advance, and decide whether to
act. Four generations of tooling could not do that at any price. CFEngine came
closest and stopped short for reasons worth being precise about, and swamp is
built for the agent that can.

## The promise theory control loop

Promise theory starts from three axioms. An agent is autonomous and cannot be
coerced. An agent can only promise its own behavior. An agent's knowledge of the
world is local. Compose them and you get a control loop:

1. **Observation.** The agent perceives its environment directly. Not from a
   file. Not from a state snapshot someone else produced. The agent looks.
   Knowledge is local, so no global observer exists to hand the agent a summary,
   and a promise is assessed by whoever observes it, so an assessment the agent
   did not make itself buys it nothing.

2. **Local reasoning.** The agent decides what to do based on what it observed.
   No central controller dictates the action. The agent's judgment is sovereign
   within its domain.

3. **Voluntary commitment.** The agent promises its own behavior. It does not
   impose behavior on others. A promise is a commitment you make about yourself,
   never an obligation you push onto a remote system. An imposition the receiving
   agent never promised to accept accomplishes nothing.

Observe, reason, commit. The loop leaves out most of promise theory, including
trust accumulation, cooperative equilibria, and the algebra of conditional
promises, and it is still enough to sort the tools.

## Terraform: diffing files against files

Terraform's agent (the `plan`/`apply` cycle) operates on two inputs: your `.tf`
files and a state file. The state file is a snapshot of what Terraform last saw,
or more precisely, what it last wrote. `terraform plan` diffs one file against
another file. It does not observe your infrastructure.

Between applies, Terraform has no awareness. If someone adds a security group
rule through the console, Terraform does not detect it until a human runs
`terraform plan`. If a Lambda configuration drifts, Terraform does not converge
toward the desired state. It sits inert until invoked.

This fails all three properties:

- **Observation:** Terraform does not observe. It reads a cached state file that
  another apply produced. The `refresh` operation partially mitigates this, but
  it runs only when a human invokes the tool. Between invocations, Terraform is
  blind. Solve that with sub-minute reconciliation loops and you have built
  something like Kubernetes. The state file records Terraform's own last write,
  the one piece of evidence an agent assessing its own promise-keeping cannot
  use.

- **Local reasoning:** Terraform does not reason. It diffs two data structures
  (desired vs. last-known) and produces a list of API calls. The reasoning was
  pre-computed by the human who wrote the HCL. Terraform executes; it does not
  decide.

- **Voluntary commitment:** Terraform does not promise its own behavior; it
  imposes changes on remote resources through API calls. The resources promise
  nothing back, and Terraform promises nothing about ongoing maintenance. It
  applies once and stops.

Terraform's actual model is closer to a batch script with a diffing preamble.
Undeniably useful, and not what Burgess described.

## Ansible: imposition from a center

Ansible's default mode inverts promise theory. A control node connects to targets
via SSH and pushes tasks. The target node has no agent, no local reasoning, no
capacity to self-assess. It receives instructions and executes them.

When the control node is down, nothing converges. Connectivity that fails mid-run
leaves operations partial with no mechanism to resume, and state that drifts
between playbook runs goes undetected.

Against the three properties:

- **Observation:** The `setup` module runs on the target, so the target does
  observe itself. It has no authority over the result. Facts travel back to the
  control node, `when:` conditions and Jinja templates evaluate there, and the
  decision returns as a task to execute. Observation without decision rights is
  telemetry rather than perception.

- **Local reasoning:** Absent. The target executes tasks in the order received.
  It does not decide whether the task is relevant, whether the action is
  necessary given current state, or whether a better path exists. It runs what
  it was told.

- **Voluntary commitment:** Inverted. The control node imposes obligations and
  the target makes no promise. In Burgess's terms this is an imposition model
  rather than a promise model, a distinction the theory is built on.

Ansible's actual model is remote procedure execution with YAML serialization.
The "agentless" marketing is technically true (no daemon on the target) and
theoretically catastrophic (no autonomous agent means no local reasoning, no
convergence, no promises).

### The honest exceptions

Ansible solved a real problem: managing devices where you cannot install an
agent. Network switches, locked-down appliances, environments where a persistent
daemon is politically or technically impossible. For those use cases, imposition
is the only available mode.

`ansible-pull` is the second exception. Run it from cron on the target and you
get a local agent that clones a playbook repository, evaluates conditions
locally, and converges on a schedule with no controller involved. That is the
promise-theoretic mode, shipped in the box, and almost nobody deploys it. So the
critique lands on how Ansible is operated rather than on what it can do. Its
dominant mode is structurally incompatible with the theory it gets associated
with, and that incompatibility costs convergence, verifiability, and autonomy.

## CFEngine: the one that kept it

Burgess built the reference implementation, and it satisfies the loop. A cf-agent
runs on every host on a short interval, five minutes by default. It observes
local state, decides locally which of its promises are unkept, and repairs those
and nothing else. No controller issues orders. A policy server publishes policy
and the agent pulls what it has agreed to apply, which is voluntary cooperation
in the literal sense Burgess meant. Take the control loop as the test and
CFEngine passes on all three counts.

Where CFEngine stops is scope and memory. A cf-agent observes what its promises
describe. Write a promise about `/etc/ssh/sshd_config` and the agent watches that
file. Write nothing about the security group in front of the host and the agent
holds no opinion, because the observation is bounded by the policy that requested
it. The agent's assessment is also a verdict rather than a record. It can tell you
whether the promise is kept right now. Ask what the config looked like last
Tuesday, or how many times this promise has been repaired in the last month, or
which hosts drifted together, and there is no queryable answer, because
promise-keeping was never stored as data.

Trust in promise theory is accumulated assessment. Accumulation needs somewhere
to accumulate.

## Chef and Puppet: the loop inside a declaration

Chef and Puppet inherited the pull-based agent and gave part of it back. A Chef
client runs on a schedule (every 30 minutes by default), pulls its run list from
a server, converges toward the declared state, and reports back. A local agent
runs autonomously and decides when to converge.

But the agent cannot observe beyond what its recipes describe. A Chef resource
checks whether a file exists, whether a package is installed, whether a service
is running. It does not survey the broader system and form a judgment about
what matters. The observation is scoped to the declaration. If you did not write
a resource for it, the agent does not see it.

The reasoning is also constrained: the agent converges toward whatever the
recipe says. It does not ask "is this recipe still relevant?" or "has the
context changed such that this action would cause harm?" It applies the
declared state mechanically.

These tools approximate promise theory within the boundaries of their
declarations, inheriting CFEngine's scope bound and adding a server dependency to
it. Outside those boundaries, they are as blind as Terraform between applies.

## What the agent actually needs

Promise theory describes an agent that can:

1. Look at a live system and capture what it finds
2. Compare what it found against what it expected (or against what existed last
   time)
3. Decide whether action is needed based on that comparison
4. Act only when the decision warrants it
5. Record what it did and what it observed, versioned over time

Steps 1 through 4 are the control loop. Step 5 is what CFEngine left out and what
the declarative generation never attempted. The gap is observational capacity and
retained assessment rather than syntax or speed. Those tools could not look, so we
looked for them, wrote our observations into files, and handed the files to tools
that could parse but not perceive.

## Swamp: observation first, action second

A model method runs against a live system:

```bash
swamp model method run my-account discover_all --json
```

This produces typed, versioned, schema-validated data. The agent observed. The
observation is stored as an immutable snapshot carrying its provenance: the model
name, id, and type, the output spec that produced it, a monotonic version, an
`isLatest` flag, a creation timestamp, and the owning run. When a workflow drove
the observation, the workflow run, job, and step names ride along too.

The next execution produces a new version. Now you can diff:

```bash
swamp data query 'modelName == "my-account" && specName == "discovery" && isLatest == true' --json
```

Reality at T compared against reality at T-1. No declaration file in the middle.
The agent queries what exists and reasons about whether the delta warrants
action.

A workflow encodes the decision logic:

```yaml
jobs:
  - name: observe
    steps:
      - name: scan
        task:
          type: model_method
          modelIdOrName: my-account
          methodName: discover_all

  - name: evaluate
    dependsOn:
      - job: observe
        condition:
          type: succeeded
    steps:
      - name: check-drift
        task:
          type: model_method
          modelIdOrName: drift-state
          methodName: compute_drift
```

The observation step runs and the evaluation step consumes its output. A
downstream step acts when drift exists, and when nothing drifted the workflow
terminates without side effects. The decision happens at workflow level rather
than inside each resource implementation.

## How the properties map

Against the three parts of the loop:

**Observation.** Model methods observe live systems directly. `discover_all`
calls AWS APIs. `cert_check` connects to a TLS endpoint. `get_os_info` shells out
to read the running kernel. The observation is the agent's own act rather than a
file a human pre-computed, and that is what makes the assessment worth anything.

**Local reasoning.** The agent (whether human or AI) queries versioned data with
CEL expressions and makes decisions. "Has this changed since baseline?" is a
query. "Should I act?" is a judgment the workflow encodes. The reasoning stays
local to the agent's context: it decides from what it observed rather than from
instructions pushed by a controller.

**Voluntary commitment.** Swamp's remote execution runs on enrollment rather than
intrusion. A worker dials out to the orchestrator, presents a scoped token, and
advertises what it can do:

```bash
swamp worker connect ws://orchestrator.internal:4000 \
  --token <name>.<secret> \
  --label region=us-east --label gpu=true
```

The orchestrator places a step only on a worker whose target name, labels, and
platform match what the step asked for. With no eligible worker the step fails
fast, and nothing lets the orchestrator reach a machine that never advertised the
capability. That is the promise-theoretic admission rule, where an imposition
lands only against a matching promise to accept. The pool is itself observable:
`swamp data query 'modelType == "swamp/worker"'`.

Burgess's model asks for more than this. Workers hold no repository and no
credentials; they execute method bodies dispatched from the orchestrator with
every capability proxied back. A worker's promise covers its
capability rather than its state, and the reasoning stays with the operator. The
loop closes around the operator's agent, and remote machines participate as
promise-keeping executors rather than as autonomous peers. That is a smaller claim
than CFEngine's per-host agent can make, and it comes with the memory CFEngine
never had.

## Built for the reasoning agent

That architecture only pays off if an agent can consume it, and a framework can
either tolerate an LLM operator or be shaped for one. The difference shows up in
small decisions repeated everywhere. Swamp is shaped for one:

- `--json` is a global option on every command, so structured output is the
  default posture rather than a flag you remember to add.
- Data carries a Zod schema and validates on write, so an agent receives typed
  fields instead of prose to scrape.
- Queries are CEL predicates over that typed data, so an agent composes a question
  rather than filtering text and hoping the format held.
- Method runs and reports emit a `narrative` alongside the structured payload, so
  one result reads correctly to a person and to a model without a second rendering
  pass.
- `swamp agent setup` lets a repository define its own agent tool definitions, so
  the repo tells the agent what it can do here instead of the agent guessing from
  a binary's help text.
- `swamp serve` exposes model and workflow execution over a WebSocket API, so an
  agent drives the system without a shell in the middle.

None of that makes a model correct. It removes the layer where agents fail hardest,
which is parsing, guessing, and inferring structure that was never guaranteed. What
remains is the part an LLM is actually good at: reading evidence and forming a
judgment about it.

A human can drive all of this by hand, and reading data queries and running
workflows manually works fine. That is no longer the case the design optimizes
for, and saying otherwise would be false modesty about what changed.

## The practical difference for a Terraform user

If you use Terraform today, here is what changes:

| Terraform | Swamp |
|-----------|-------|
| State file records last-known state | Versioned data records observed state at every snapshot |
| `plan` diffs desired vs. last-known | Queries diff reality at T vs. reality at T-1 |
| Blind between applies | Scheduled observations record reality on every run |
| Refresh is opt-in and synchronous | Observation is the primary operation |
| One state file per workspace | Queryable datastore across all models and domains |
| HCL declares what should exist | Methods observe what does exist; workflows decide whether to act |

The shift: you stop writing declarations that tell a blind tool what to do. You
start defining observations that let the agent see for itself.

## The practical difference for an Ansible user

If you use Ansible today, here is what changes:

| Ansible | Swamp |
|---------|-------|
| Central controller pushes tasks via SSH | Local agent queries and reasons; connects only when acting |
| Targets have no agency | Observation data makes target state visible without a daemon |
| No convergence between runs | Scheduled workflows detect drift and can correct it |
| Playbook reports "last push succeeded" | Data query answers "is my fleet in the desired state right now?" |
| Idempotency is per-module | Idempotency is a workflow decision based on observed state |
| `gather_facts` runs at push time | Observations run on schedule and persist as versioned data |

Here the shift runs from pushing obligations outward and hoping they arrive intact
to observing state continuously and acting only when the observation warrants it.

## The observer has an availability problem too

A workflow can carry a `trigger.schedule` cron expression, and swamp fires it only
while `swamp serve` is running. Schedules missed while serve was down do not catch
up on startup. So the objection I aimed at Ansible's control node lands here as
well: when the observer is down, nothing observes.

What differs is the cost of a gap. A missed Ansible push leaves an intended change
unmade, unrecorded, and invisible until someone pushes again. A missed observation
leaves a hole in a record that is otherwise queryable, and the next run still sees
current reality and still diffs it against the last stored version. CFEngine
handles this better than either by putting the interval on the host, where nothing
central has to be up. Anyone who wants that property today can run the schedule
from the host's own cron and point it at a shared datastore.

## Idempotency moves up

Both Terraform and Ansible require idempotency at the resource/module level.
Every `aws_security_group_rule` must detect whether it already exists. Every
Ansible module must check whether the file is already in the desired state.

When the agent can query versioned state before executing, idempotency becomes a
property of the workflow's judgment. The agent asks: "has this already been
done?" It checks the latest observation. If the answer is yes, it skips. The
individual method does not need convergence logic because the workflow decided
not to invoke it.

Swamp spells this as a step `guard`, a CEL expression evaluated before the step
runs. A truthy result skips the step and records the reason as `guarded`:

```yaml
- name: create-instance
  guard: ${{ data.latest("my-instance", "resource") }}
  task:
    type: model_method
    modelIdOrName: my-instance
    methodName: create
```

A guard can also call a method to probe live state when stored data is not enough
to decide. For a scheduled workflow, guards are the difference between converging
and re-running everything every thirty minutes.

Terraform pushes idempotency down because it cannot reason at a higher level.
Ansible pushes idempotency down because it has no persistent state to query. Once
persistent, versioned, queryable state exists, the decision moves up to the
judgment layer where it belongs.

## Our thinking moves up with it

Idempotency is the easiest instance to point at and the least interesting one. The
same movement runs through the whole practice. We used to hand-write the
convergence check, the retry, the ordering, the drift detection, and the
reconciliation, one resource at a time, because no tool could be trusted with any
of it. Give an agent observed state and a way to reason over it, and all of that
drops into machinery. What remains for us is the altitude above it: what should be
true, what evidence would show it, what is worth acting on, and what a system is
entitled to promise about itself.

That is a smaller job by volume and a larger one by consequence, and it still needs
a vocabulary. Promise theory earns its keep by supplying the words for exactly the
questions that survive automation. Who is the agent here? What does it observe, and
can it observe that directly? What may it promise, and about whom? Where does an
imposition happen, and did anything on the receiving end agree to accept it? Those
questions do not resolve into YAML, and no higher-level tool retires them. They get
sharper as the machinery gets more capable, because more behavior is in flight and
less of it sits under anyone's direct hand.

## Where declarations still matter

Three cases require stated intent, and swamp does not pretend otherwise:

**Provisioning.** You cannot observe what does not exist. Creating a new VPC
requires specifying what you want before reality contains it. But provisioning is
a small fraction of infrastructure work. Most of the work is managing what
already exists.

**Compliance baselines.** "All S3 buckets must have encryption enabled" is
intent, not observation. That intent lives in a Zod schema or a workflow
condition rather than a per-resource YAML block, and it remains a declaration.

**Rollback targets.** "Restore Tuesday's state" requires knowing what Tuesday
looked like. Versioned data provides this, but choosing the target version is a
human decision.

In all three cases, the intent lives in method logic and schema definitions rather
than in static files a blind tool reads. They remain declarations in a different
medium, over a much smaller surface.

## The 30-year gap

Burgess shipped an agent that observed, reasoned, and promised in 1993. The
missing piece was never the infrastructure agent, which could have been built at
any point in the three decades since. It was the reasoning agent, the one that can
look at an observation, compare it against context nobody programmed it to
anticipate, and decide whether to act.

The credit for closing that gap belongs outside infrastructure, which is worth
sitting with rather than glossing. Nothing in three decades of declaration formats
was converging on a reasoning agent, and no amount of further refinement in that
lineage would have produced one. The capability showed up as a general one and
infrastructure inherited it.

Terraform cannot supply reasoning, since a state file is memory of a past action
rather than perception, and neither can Ansible, whose playbook is a script pushed
from a center. CFEngine had the loop and nowhere to keep what it learned.

The promise was always about agents that observe and decide, and we finally have
agents that can do both. The infrastructure layer should get out of their way.

## Getting started

If you have Terraform experience, the translation is:

```bash
# Install swamp
# https://swamp.club/docs/install

# Pull an AWS observation extension
swamp extension pull @webframp/aws/adopt

# Create a model instance (like defining a provider plus backend)
swamp model create @webframp/aws/adopt my-account \
  --global-arg profile=my-profile \
  --global-arg region=us-east-1

# Observe (like terraform refresh, but the primary operation)
swamp model method run my-account discover_all

# Query what you found (no equivalent in Terraform)
swamp data query 'modelName == "my-account" && isLatest == true' --json
```

If you have Ansible experience:

```bash
# Pull an SSH fleet extension
swamp extension pull @swamp/ssh

# Create the fleet definition, then fill in transport and hosts
swamp model create @swamp/ssh fleet --json
swamp model edit fleet

# Observe across the fleet (like gather_facts, and the output persists)
swamp model method run fleet exec \
  --input hosts=all --input command='uname -a'

# Query across all observations
swamp data query 'modelName == "fleet" && isLatest == true' --json
```

The fleet definition is a declaration of where to look, and it says nothing about
what should be true once you get there. That boundary is the whole argument.

Instead of writing a playbook that pushes state, you define observations that
persist, version, and compose. Action follows observation, so the tool sees before
it touches.

## The concepts outlive the tools

The value of promise theory was never the toolchain it produced. CFEngine was one
implementation. Chef and Puppet were partial ones. Terraform and Ansible were not
implementations at all, whatever their marketing borrowed. Swamp is another
attempt, and something will supersede it. What persists across all of them is a way
to reason about administering systems, which is a different kind of asset than a
product.

So the honest use of a framework this durable is to hold the concepts and let the
rest go. Keep autonomy, locality of knowledge, promises over impositions, and
assessment by whoever observes. Give up the state file, the playbook, the run list,
the per-resource convergence check, and the reflex of declaring what should be true
to a tool that cannot look. Thirty years of tooling accumulated around a genuinely
good idea, and most of that accumulation was scaffolding for capabilities the tools
did not have. The scaffolding was the right call at the time.

Burgess was right about what agents should do, and he proved it in 1993. What took
thirty years was an agent that could reason about what it saw, and a place to keep
what it learned. Both showed up at once, and neither came from the direction anyone
was watching.
