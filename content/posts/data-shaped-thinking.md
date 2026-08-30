---
title: "Designing for the Query"
date: 2026-06-01T10:00:00-04:00
tags: [swamp, infrastructure, devops, ai]
description: "What changes when you design for data-out instead of declaration-in"
draft: true
---

The hardest thing to explain about swamp is not the syntax. It is the moment
where your instincts betray you.

Every tool you have used — Chef, Puppet, Ansible, Terraform — trained you to
think in named things. One resource per block. One recipe per node. One playbook
per host group. You have been practicing that reflex for a decade or more.

When I walk someone through
[@swamp/ssh](https://swamp.club/extensions/@swamp/ssh), their first question is
almost always "so I create one model per host?" The reflex is immediate. The
answer is no. You create one model for the fleet. Methods target hosts by
selector. Data accumulates per-host within a single collection. And the moment
that clicks, the person is not learning a new tool anymore — they are unlearning
a design assumption they did not know they had.

## What declarations got right

Before I invert the model, declarations deserve their credit.

A Terraform resource block is a remarkable piece of communication. In ten lines
of HCL, a human who has never seen your infrastructure can understand what
exists, how it is configured, and what depends on what. The declaration is
self-documenting. It is reviewable in a pull request. It is diffable. It
compresses an enormous amount of operational context into a format that humans
can scan and reason about quickly.

```hcl
resource "aws_instance" "web_1" {
  ami           = "ami-0c55b159cbfafe1f0"
  instance_type = "t3.micro"
  tags = { Name = "web-1" }
}

resource "aws_instance" "web_2" {
  ami           = "ami-0c55b159cbfafe1f0"
  instance_type = "t3.micro"
  tags = { Name = "web-2" }
}
```

Chef recipes had the same virtue. You could read a cookbook and understand the
system it produced. Puppet manifests were legible to anyone who knew the
language. The declaration was optimized for human comprehension at authoring
time, and that was the right optimization when humans were the only readers that
mattered.

The problem is not that declarations are bad. The problem is that the reader
changed.

## The reader changed

When an agent needs to reason about your fleet, it does not open a `.tf` file
and scan for resource blocks. It queries structured data. The state file
Terraform produces is a flat list of resources keyed by block address.
`aws_instance.web_1` and `aws_instance.web_2` are siblings in a list. To ask a
question across them — "which instances are running t3.micro?" — you parse the
state file and filter. The state was designed to track individual resources, not
to answer questions about collections.

This was fine when the only consumer of that state was `terraform plan`. But
agents consume state now. Workflows compose across it. CEL queries run against
it. The question is no longer "can a human read this declaration?" It is "can an
agent query this data?"

That is the shift. Not from one syntax to another, but from designing for the
human who authors to designing for the agent who queries.

## The data-shaped mind

The [@swamp/ssh](https://swamp.club/extensions/@swamp/ssh) extension inverts
this — and it is not unique; swamp in general rewards the inversion. One model
instance represents the entire fleet. The fleet declares its hosts once in
`globalArguments`, then every method targets subsets by selector. Data
accumulates per-host within a single versioned collection.

```yaml
# One model instance for the fleet — hosts declared once
name: prod-fleet
transport:
  kind: ssh
  user: deploy
  identityFile: ~/.ssh/fleet_ed25519
  controlMaster: { enabled: true, persistSec: 600 }
hosts:
  - name: web-1
    address: web-1.prod.example.com
    tags: [web, prod]
  - name: web-2
    address: web-2.prod.example.com
    tags: [web, prod]
  - name: db-1
    address: db-1.prod.example.com
    tags: [db, prod]
```

Methods target hosts by selector argument — a tag, a name list, a CEL
expression, or `all`:

```bash
# Run against every host tagged 'prod'
swamp model method run prod-fleet exec \
  --input hosts='tag:prod' \
  --input command='uptime' --json

# Run against a specific host
swamp model method run prod-fleet exec \
  --input hosts='name:db-1' \
  --input command='pg_isready' --json

# CEL predicate for complex selection
swamp model method run prod-fleet exec \
  --input hosts='cel:"web" in host.tags && host.attrs.region == "us-east-1"' \
  --input command='systemctl status nginx' --json
```

Each method run writes a `run-<method>-<host>` resource per matched host. After
a week of operations, the model contains typed, versioned run results for every
host, every method, every invocation — all queryable from a single expression:

```bash
swamp data query 'modelName == "prod-fleet" && isLatest == true' --json
```

One query returns the current state of every host in the fleet, filterable by
any field in the schema, comparable across versions to surface drift.

## The factory pattern

This is the factory pattern: the model represents a collection, methods operate
on targets by argument. The model is not "web-1" — the model is "prod-fleet."
The method is not "run a command on this one host" — the method is "run a
command on whichever hosts match this selector."

The distinction matters because it changes what the data layer looks like. In
the one-model-per-host pattern, you get:

```
ssh-web-1/  → runResults: [v1, v2, v3]
ssh-web-2/  → runResults: [v1, v2]
ssh-db-1/   → runResults: [v1]
```

In the factory pattern, you get:

```
prod-fleet/ → resources: {
  "run-exec-web-1": versions [v1, v2, v3],
  "run-exec-web-2": versions [v1, v2],
  "run-exec-db-1": versions [v1],
  "host-web-1":    versions [v1],
  "host-web-2":    versions [v1],
  "host-db-1":     versions [v1]
}
```

Same hosts. Radically different queryability. The factory pattern produces a
collection you can reason about as a unit — hosts, run results, master audit
events, forward state, all colocated under one model with gc policies that
retain history (50 run results per host by default). The per-host pattern
produces isolated atoms you must reassemble.

## What changes in practice

**Schemas come first.** In Terraform, you start with "what resource do I need?"
and the schema is whatever the provider defines. In swamp, you start with "what
data do I want to query later?" and the schema is your first design decision.
The `@swamp/ssh` extension defines a `runResult` schema that captures exit code,
stdout, stderr, timing, and the command that produced them. That schema
determines what questions you can answer about your fleet's operational history.
If the schema did not include timing data, you could not query for hosts where
commands are taking longer than expected.

**Instance names lose their weight.** In Terraform, naming the resource block is
a design decision that affects the state file forever. Rename
`aws_instance.web_1` to `aws_instance.web_primary` and Terraform wants to
destroy and recreate it. In the factory pattern, the instance name is decided
once (the fleet name) and the selector determines which targets to operate on.
Adding a new host to the fleet is a line in `globalArguments.hosts` and an
`apply` call — not a new model instance that restructures the data layer.

**Fleet-level questions become native.** "Which hosts failed their last exec?"
is a CEL expression against the collection, not a loop over thirty individual
state files:

```bash
swamp data query \
  'modelName == "prod-fleet" && specName.startsWith("run-exec-") && data.exitCode != 0 && isLatest == true' \
  --json
```

When you design the model as a factory, the collection-level question is the
natural one. When you design one model per host, the same question requires
aggregation that the data layer does not provide.

## The boundary that dissolves

I keep coming back to a Lisp principle when I think about this shift. In Lisp,
code and data share the same representation — a property called
[homoiconicity](https://en.wikipedia.org/wiki/Homoiconicity).[^1] The boundary
between "the thing you write" and "the thing the machine operates on" dissolves.

Traditional IaC maintains that boundary rigidly. The declaration (HCL, YAML,
Ruby) is what you write. The state file is what the tool operates on. Different
formats, different structures, different mental models. You reason in
declarations. The tool processes state. Translation happens at `terraform plan`.

In swamp, the method execution and the data production are the same operation.
When `@swamp/ssh`'s `exec` method runs against `tag:prod`, it spawns SSH
processes in parallel, captures their output, and writes one `runResult`
resource per host — typed, schema-validated, versioned, queryable. No
intermediate declaration file. No state translation step. The code that observes
is the code that produces the data that informs the next decision.

This matters practically because it eliminates an entire class of drift. In
Terraform, the declaration can say one thing while reality says another, and you
discover the gap at `plan` time. In swamp, the data *is* the observation. There
is nothing to drift from because there is no separate declaration to drift
against. The agent that ran the method and the agent that queries the result
operate on the same typed structure — one representation serving both roles, the
way a Lisp S-expression is simultaneously the program and the data the program
manipulates.

## Where declarations survive

The [previous post](/posts/you-were-never-declaring-state/) identified three
cases where stated intent remains necessary: provisioning, compliance baselines,
and rollback targets. The data-first model does not eliminate these. It relocates
them.

In Terraform, the declaration is the primary artifact. Everything flows from the
`.tf` file. In swamp, the declaration is a constraint embedded in method logic
or workflow conditions. "All hosts must respond to health checks within 5
seconds" is not a resource block — it is a CEL expression evaluated against the
fleet data:

```bash
swamp data query \
  'modelName == "prod-fleet" && specName.startsWith("run-exec-") && data.durationMs > 5000 && isLatest == true' \
  --json
```

The compliance baseline lives in the query, not in a declaration file. The data
is primary, the constraint operates on it, and the declaration moved from input
to evaluation.

## The design question that remains

When you sit down to model a new domain in swamp, the first question is no
longer "what resources do I need to declare?" It is "what data do I want to
accumulate, and what questions do I want to ask of it?"

That question determines whether you create one model per thing (the Terraform
instinct) or one model per collection with methods that target by argument (the
factory pattern). From that choice, the schema follows, the versioning
granularity follows, and the difference between an agent that reasons about the
domain as a whole versus one that reasons about individual atoms follows.

Your declarations were always shaped for a human reader. The human reader is no
longer the bottleneck. The agent querying your data at 3 AM to decide whether
your fleet needs intervention — that is the reader now. And that reader does not
care how pretty your resource blocks are. It cares whether your data answers its
questions.

Are you still designing for the reader who left the room?

[^1]: The term comes from Lisp's structural identity between its code and its
    data — both are S-expressions. The practical consequence: programs that
    manipulate other programs are trivial to write because there is no format
    translation between "the thing you execute" and "the thing you inspect."
