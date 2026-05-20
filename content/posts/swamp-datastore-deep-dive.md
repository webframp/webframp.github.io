---
title: "Swamp Datastores: One Workflow, Any Machine"
date: 2026-05-20T10:00:00-04:00
tags: [swamp, datastore, workflow, ai]
description: "Agent state that follows you, then your team"
---

By default, swamp stores everything in local SQLite. This works until you close
your laptop, sit down at your desktop, and discover that your agent has no memory
of what happened an hour ago. The models are there (they live in YAML), but the
runtime data, the execution history, the versioned outputs from every method run,
all of it is trapped on whichever machine produced it.

If you work on one machine, this is fine. If you work on two or three, it is a
problem. Your agent starts cold every time you switch. The voice profile you
refined yesterday on your laptop does not exist on your desktop. The health check
workflow that ran at 03:00 on your VM has results you cannot see from anywhere
else. You are running the same models against the same code, but the data layer
is fragmented across physical locations.

The swamp datastore primitive exists to fix this.

## What a datastore is

A datastore is a pluggable backend that replaces local SQLite storage with a
remote, shared data layer. The [provider interface](https://github.com/systeminit/swamp/blob/main/src/domain/datastore/datastore_provider.ts)
defines four capabilities:

- **createLock**: acquire and release distributed locks so concurrent sessions do
  not corrupt state
- **createVerifier**: health check the backend before trusting it
- **createSyncService**: pull remote data to a local cache, push local changes back
- **resolveDatastorePath / resolveCachePath**: tell swamp where the data lives

Any extension that implements this interface becomes a valid backend. Swamp does
not care whether the data sits in PostgreSQL, S3, or a GitLab project. It calls
the same methods regardless.

Configuration lives in `.swamp.yaml`. Here is what a GitLab backend looks like
(more on this one below):

```yaml
datastore:
  type: "@webframp/gitlab-datastore"
  config:
    projectId: "12345"
    baseUrl: "https://gitlab.com"
    token: "glpat-xxxxxxxxxxxxxxxxxxxx"
    statePrefix: "swamp"
```

Once configured, the CLI handles sync:

```bash
# Check backend health
swamp datastore status

# Pull remote state into local cache
swamp datastore sync --pull

# Push local changes to remote
swamp datastore sync --push
```

That is the entire user-facing surface for multi-machine continuity. Pull before
you start, push when you finish. Your agent resumes where it left off regardless
of which machine you are on. For a complete example of standing up a remote
datastore from scratch, the upstream
[s3-bootstrap workflow](https://github.com/systeminit/swamp-extensions/tree/main/workflows/s3-bootstrap)
walks through provisioning, configuring, and verifying an S3 backend in one run.

## Two backends, one interface

I built two datastore extensions to cover different deployment scenarios. Both
implement the same provider interface. Both solve the same problem. They differ
in what infrastructure you already have.

### GitLab: the infrastructure you already own

The [@webframp/gitlab-datastore](https://github.com/webframp/swamp-extensions/tree/main/datastore/gitlab-datastore)
stores swamp data using GitLab's Terraform state HTTP API. Every piece of swamp
data becomes a Terraform state object in your GitLab project.

The fit is natural. GitLab's state API provides exactly what a datastore needs:
content-addressable storage, native locking, serial number tracking, and an API
that already exists in every GitLab project without additional setup. The
extension wraps swamp data in a Terraform state envelope, handles serial
increment automatically, and uses GitLab's lock endpoint for distributed locking.

If you already have a GitLab project (and many teams do), you have a swamp
datastore. No database to provision. No S3 bucket to create. No additional
infrastructure at all.

### PostgreSQL: when you need production guarantees

The [@webframp/postgres-datastore](https://github.com/webframp/swamp-extensions/tree/main/datastore/postgres)
stores swamp data in PostgreSQL with row-based distributed locking. It targets
AWS RDS, Aurora, and Aurora Serverless v2.

The locking strategy here is different from GitLab. PostgreSQL
[advisory locks](https://www.postgresql.org/docs/current/explicit-locking.html#ADVISORY-LOCKS)
do not survive Aurora failover (they exist in shared memory, not in the WAL). So this
extension uses row-based locks with fencing tokens and heartbeat-based TTL. A
crashed process leaves a stale lock. The heartbeat stops. The TTL expires. The
next session reclaims it automatically.

```yaml
datastore:
  type: "@webframp/postgres-datastore"
  config:
    connectionString: "postgres://user:pass@your-rds-cluster:5432/swamp"
    schema: "swamp"
    ssl: "verify-ca"
    sslCaPath: "/path/to/rds-global-bundle.pem"
```

This backend is for teams that need durability guarantees, audit trails in a real
database, and the operational tooling that comes with RDS (backups, monitoring,
failover).

## The pattern for a single developer

The simplest deployment is one person with two machines and a GitLab project.

On the first machine:

```bash
swamp extension pull @webframp/gitlab-datastore
# Configure .swamp.yaml with your GitLab project
swamp datastore status
swamp datastore sync --push
```

On the second machine:

```bash
swamp extension pull @webframp/gitlab-datastore
# Same .swamp.yaml configuration
swamp datastore sync --pull
```

Every model, every version of every data artifact, every workflow execution
result is now available on both machines. The agent does not need to re-derive
anything. It picks up where the other session left off.

The `.swamp.yaml` file commits to your repo. The credentials go in a swamp
vault. The data flows through the configured backend. When you switch machines,
you run one command and your agent has full context.

## From one machine to a team

The single-dev pattern is the starting point, not the ceiling. The same
datastore that syncs between your laptop and desktop works identically when a
second person connects. Their agent pulls the same versioned state. Their
workflow runs produce data visible to yours. The distributed locking that
prevents your own concurrent sessions from conflicting prevents theirs too.

Nothing changes architecturally. A team of five pointing at the same PostgreSQL
backend or the same GitLab project shares a single source of truth for all agent
state. Workflows one person creates produce outputs another person's agent can
query. The voice profile one person refines is immediately available to everyone
else. Models stay in version control. Data flows through the shared backend. The
gap between "I use swamp" and "my team uses swamp" is one configuration change.

## Why this matters

The default local-only datastore is not a limitation of swamp. It is a
reasonable default for getting started. But agents accumulate state. Every model
method run produces versioned data. Workflows compose those outputs. Reports
summarize them. Over days and weeks, the local SQLite database becomes a record
of everything your agent has learned and produced.

Fragmenting that record across machines means fragmenting your agent's context.
A datastore extension makes the record portable. Your agent's history follows
you or your team, not your hardware.

The provider interface is [open](https://github.com/systeminit/swamp/blob/main/src/domain/datastore/datastore_provider.ts).
GitLab and PostgreSQL are two implementations. An S3 backend exists upstream from
[@swamp/s3-datastore](https://github.com/systeminit/swamp-extensions/tree/main/datastore/s3).
The interface is small enough that a new backend is a single TypeScript file
implementing four methods.

If you have a place to store bytes and a way to lock a resource, you have a
swamp datastore.

## The opt-in adoption curve

None of this is required on day one. The local SQLite default is the right
starting point. You install swamp, create models, run workflows, and everything
works on a single machine with zero configuration.

When you outgrow that, you add a datastore line to `.swamp.yaml`. Your existing
models, workflows, and extensions do not change. The data just flows somewhere
else. You do not refactor. You do not migrate. You opt in.

The progression looks like this:

1. **Single machine, local SQLite.** Learn the primitives. Build models. Run
   workflows. No infrastructure decisions yet.
2. **Multiple machines, shared datastore.** Add a GitLab or S3 backend. Push and
   pull between sessions. Same workflows, portable state.
3. **Team adoption, shared backend.** Point the team at a PostgreSQL instance or
   a shared GitLab project. Distributed locking handles concurrency. Everyone's
   agent reads from the same versioned data.

Each step is one configuration change. No architectural rework. No rewrite of
existing automations. The tool grows with you because the datastore is a layer
underneath everything else, not a decision you make upfront that constrains what
comes later.

At step three, something new becomes possible. A shared datastore means CEL
queries run against everyone's data. Your agent can ask "show me the latest
health check results from any team member's workflow run" and get an answer,
because all that versioned output lives in one place. The data layer becomes
queryable team memory, not just portable state.
