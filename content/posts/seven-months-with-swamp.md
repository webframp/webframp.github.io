---
title: "Seven Months with Swamp"
date: 2026-10-14T10:00:00-04:00
tags: [swamp, ai, agents, devops, infrastructure]
description: "I tried agents with skills. When they failed, I found swamp."
---

*Swamp launches officially today. The team asked whether I'd write
something for the launch, and I said yes on the condition that it be what I
actually think. I'm a user, a community contributor, and a paying customer.*

## Best effort

I was trying to get an agent to work with the infrastructure tools I already
had. CloudFormation ran some stacks and Terraform ran others. I wanted an agent
in the middle to plan changes, apply them, and tell me what happened.

It kept failing. Seven months later I couldn't tell you which run failed on
which day, but I remember the shape of it. Everything the agent knew about
driving those tools lived in skills, prose instructions it followed on a
best-effort basis. The tools underneath were deterministic. The layer driving
them wasn't, and every fix I made went into the prose, where the next run was
free to ignore it.

## Three alarms

In April, a month into using swamp, I ran an outage investigation without a
workflow. The agent checked three alarms, declared the system healthy, and
never pulled the metrics where a cold-start spike was sitting. I wrote about it
in [Building the Machine](/posts/building-the-machine/). Once a workflow pulled
those metrics on every run, the agent stopped missing them, and I stopped
hoping it would remember.

Swamp moves the procedure out of the prompt and into code. A model is a typed
schema with methods that run against a live system and store what they find as
versioned data. Workflows compose those methods into a DAG. The agent no longer
has to remember how to run Terraform correctly, or which metrics to pull. It
decides which
behaviors to run and reads what they produce. The team calls this
behavior-centric design: instead of improving outcomes by improving the agent,
you put the behavior in code the agent calls.

With a skill, the agent decides whether to follow the procedure. With a model
method, the procedure is code: you can version it, test it, and review it, and
it runs the same way after the next model upgrade. Most of what I've written
here since April, from
[You Were Never Declaring State](/posts/you-were-never-declaring-state/) to
[The Promise None of Them Kept](/posts/the-promise-none-of-them-kept/), is
about that difference.

## Zero API calls

What kept me building was the way the work compounded. Early on I wrote an
`adopt` extension that observes VPCs, subnets, security groups, and RDS
clusters. It manages nothing. Each run records what's live as a new version,
and the older versions stay. Later I wrote observers for inventory, config
compliance, DNS, and event topology, each for its own reason and none
referencing the others.

In June I wrote a drift detector that makes zero AWS API calls of its own. The
observers make the calls. The drift detector reads the newest version each one
recorded, normalizes it, and diffs it against a baseline. It found the security
group rule I added, the Lambda timeout I changed, and the EventBridge target
that hadn't existed at baseline. [The whole
thing](/posts/drift-detection-as-composition/) is normalizers and a recursive
diff. Everything else was already there. I didn't plan for drift detection when
I wrote `adopt`.

As of this week I've published 165 extensions to the registry. The largest
groups cover Cloudflare, AWS, Snyk, and Datadog. The rest include image
generation, a Twitch integration, and a monthly reconciliation of our Claude
invoice against Anthropic's usage API. The source lives at
[webframp/swamp-extensions](https://github.com/webframp/swamp-extensions/).

## Five hundred workers

In July I wanted to know whether a swamp datastore could survive a swarm. I
had an agent design a load test, pointed it at ECS Fargate, and stepped away.
It climbed to 500 workers and pushed roughly triple swamp's lifetime telemetry
volume through its ingest pipeline in a couple of hours, enough to overwhelm it
for about three minutes. Outside that window, nobody would have known. Nick
from swamp caught it before I did. When I offered to stop, he told me to keep
going because he wanted to see the ceiling. Later he asked for a pause so the
backlog could drain, and told me it was the best kind of thing that could have
happened. I wrote about [what that said about trust](/posts/six-million-events/),
and he wrote
[his side of it](https://keeb.dev/2026/07/22/three-minutes-of-trouble/).

The same month I spent four rounds fixing a DynamoDB sync backend. Each fix
moved the ceiling somewhere else: a hot GSI partition, then lock contention,
then catalog metadata that no datastore could outrun.
[The Migrating Bottleneck](/posts/the-migrating-bottleneck/) is the long
version.

I found both problems by pushing harder than most users will, which is the
only way I know to learn where a system's ceiling is.

## A hundred engineers

I got to "this solves my problem" within the first week. The question I care
about now is what happens when more than a hundred engineers share the same
models, datastores, and workflows.

Much of that is already in place. Moving from a local repository to a [shared
datastore](/posts/swamp-datastore-deep-dive/) is one configuration change. The
[collision between a team's workflow and an agent's
lifecycle](/posts/the-workflow-collision/) has a workable shape once you let
one run inside the other. The July load tests asked the team question early: I
wanted to know whether a datastore would hold up under that much write pressure
before a team that size depended on it. Many teams already run swamp, and
they're writing the playbook for moving from one person's repository to shared
practice as they go. I'd like to add what I learn at this scale.

The hardest part may be explaining why it's worth doing. The best explanation
I have is the drift detector, and it only lands with people who have maintained
a state file. Everyone else hears "it diffs two snapshots" and wonders why that
needed a framework.

Seven months in, swamp is the first thing I reach for, and not only for
infrastructure. If you've run agents against last generation's tools and
watched them follow instructions most of the time, it's worth your time.
Start at [swamp-club.com](https://swamp-club.com/) and write one model that
observes something you care about. When the second question shows up, the data
for it will already be there.
