---
title: "The Pipeline Is Dead, Long Live the Agent Mesh"
date: 2026-05-21T10:00:00-04:00
tags: [devsecops, ai, swamp, workflow, security]
description: "Sequential pipelines assume humans waiting in sequence. Agents do not wait."
---

Picture three agents running concurrently against the same repository. One is
implementing a feature. One is scanning the dependency tree for known
vulnerabilities. One is validating that the infrastructure change will not
violate the compliance baseline. They share typed state through a common data
layer. When the security agent finds a vulnerable transitive dependency, the
implementation agent sees that result immediately via a CEL query and adjusts its
import before the PR is even open.

No queue. No handoff. No ticket filed in a separate system that someone triages
next Tuesday.

This replaces your pipeline.

## What the pipeline assumed

The traditional pipeline is a line. Code moves through stages: build, test, scan,
review, approve, deploy. Each stage has an owner. Each transition is a handoff.
The security scan happens after the build. The change advisory board meets after
the security scan. The environment promotion happens after the CAB approves.

This made sense when each stage required a different human with different context
to make a judgment call. The pipeline encoded organizational boundaries into
infrastructure. It was not an engineering decision. It was a Conway's Law
artifact.

I have been on both sides of this. I have waited three days for a security review
that took fifteen minutes of actual work. I have been the person whose approval
queue grew faster than I could clear it, knowing that every item in that queue
was blocked work for someone else. The pipeline creates wait time. Wait time is
the dominant cost in most software delivery processes, not execution time.

## What agents change

Agents do not wait in queues. They execute when they have sufficient context. If
the security context is available as typed data, the agent consumes it at the
moment it is relevant, not at a fixed stage in a fixed sequence.

This is the shift: from sequential stages with handoffs to concurrent agents with
shared state. Paul Stack's description of
[the swamp issue lifecycle](https://stack72.dev/the-lifecycle-of-a-swamp-issue/)
shows the prerequisite: every meaningful step writes typed data the next step can
read. Once that infrastructure exists, the question becomes which steps actually
depend on each other and which were only sequential because humans were.

In swamp, this looks like a workflow where jobs run in parallel, not in series:

```yaml
jobs:
  - name: implement
    steps:
      - name: generate-code
        task:
          type: model_method
          modelIdOrName: feature-agent
          methodName: implement
          inputs:
            issue: ${{ inputs.issueId }}
    dependsOn: []

  - name: security-scan
    steps:
      - name: dependency-check
        task:
          type: model_method
          modelIdOrName: security-scanner
          methodName: scan-dependencies
          inputs: {}
    dependsOn: []

  - name: compliance-check
    steps:
      - name: baseline-verify
        task:
          type: model_method
          modelIdOrName: compliance-baseline
          methodName: verify
          inputs: {}
    dependsOn: []

  - name: synthesize
    steps:
      - name: gate
        task:
          type: model_method
          modelIdOrName: release-gate
          methodName: evaluate
          inputs: {}
    dependsOn:
      - job: implement
        condition:
          type: succeeded
      - job: security-scan
        condition:
          type: succeeded
      - job: compliance-check
        condition:
          type: succeeded
```

Three jobs start simultaneously. The fourth job (synthesize) depends on all three
completing. The security scan does not wait for implementation to finish. The
compliance check does not wait for the security scan. They share a data layer and
can query each other's outputs as they appear.

The synthesize step reads all three results and makes a single go/no-go decision
with full context. It is not a human in a meeting. It is a typed evaluation
against versioned criteria.

## What breaks

Four things that exist because of sequential assumptions:

**Change advisory boards.** A CAB exists to aggregate risk assessment from
multiple domain experts into a single approval decision. If each domain expert
is an agent that produces a typed, versioned risk assessment, the CAB is a CEL
query:

```bash
swamp data query 'modelName == "release-gate" && isLatest == true' --json
```

The data is the meeting. The query is the vote.

**Security review queues.** [Little's Law](https://en.wikipedia.org/wiki/Little%27s_law)
tells you that wait time is proportional to queue depth. A security review queue
grows because human processing capacity is fixed while arrival rate is not.
Agents eliminate the queue entirely by running against every PR concurrently. No
queue, no wait time. The constraint that created it (limited human attention) no
longer applies.

**Environment promotion gates.** A promotion gate says "this artifact was
approved for staging, now it needs separate approval for production." The gate
exists because the staging approval and the production approval require different
context that was historically held by different people. If both contexts are
typed data in the same datastore, the production evaluation can reference the
staging results directly. No separate ceremony.

**Sequential handoffs between concerns.** Even in teams that practice "you build
it, you run it," security and compliance review often remain serialized. The team
owns the full lifecycle, but the security scan still blocks the deploy. The
compliance check still runs after the scan. In an agent mesh, these concerns
execute concurrently because they do not depend on each other's output. The team
still owns everything, but the work that used to happen in sequence now happens
in parallel. Ownership does not change. Timing does.

## The organizational design underneath

The agent mesh does not eliminate roles. It changes what roles produce.

A security team in a pipeline world produces approvals. In an agent mesh world,
they produce typed security context: vulnerability databases, compliance
baselines, threat model schemas, policy rules. They publish swamp extensions that
other agents consume. Their output is infrastructure, not decisions.

This maps to the platform team pattern. The security team becomes a platform team
whose product is security-as-data. Stream-aligned teams consume that data through
their agents automatically. The interaction mode shifts from "request and wait"
to "consume and verify."

The cultural prerequisite is trust. An organization where security context flows
freely requires an organization where information flows freely. Where failure is
investigated, not punished. Where the default is shared context, not hoarded
expertise. Without that, the agent mesh becomes another system that leadership
locks down because they do not trust the people using it.

## What this looks like in practice

Today, in swamp, I run workflows where security scanning and implementation
happen in the same DAG. A concrete example:
[@swamp/cve/dirtyfrag](https://swamp.club/extensions/@swamp/cve/dirtyfrag) scans
hosts for Linux privilege escalation vulnerabilities using nothing but POSIX
utilities and procfs. Its `scanFleet()` method runs in parallel across targets,
produces structured status data, and requires no agent or package on the remote
host. That scan runs concurrently with implementation and compliance jobs in the
same workflow. The compliance model queries dirtyfrag's output directly. The
implementation model can act on findings before the workflow even reaches a
synthesis step.

Three extensions, one workflow, and a shared datastore. Already running.

The pipeline is dead because its assumptions are. Agents do not wait in
queues. They do not hand off context through tickets. They do not respect stage
boundaries that exist to protect human attention spans. They read typed data and
act on it immediately.

The only question left: is your organization ready to publish its security
context as data instead of hoarding it behind an approval queue?
