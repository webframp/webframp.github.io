---
title: "Building the Machine"
date: 2026-04-26T18:35:00-04:00
tags: [devops, ai, aws, swamp]
description: "Encoding judgment the agent lacks into AWS operations"
draft: false
---

I wrote recently about the difference between easy and simple.[^1] That post
was philosophical. This one is not. This is what it looks like in practice.

I have been building an AWS operations toolkit using
[Swamp](https://swamp.club). The toolkit investigates outages. It runs daily
health checks. It composes nine separate extensions into two workflows that
gather data from CloudWatch, X-Ray, EC2, Lambda, and load balancers, then
produce an actionable report.

This is the machine that builds the machine. Not the code. The system that
produces the code and makes sense of the output.

## The problem

Incident investigation is repetitive. Check the alarms. Pull the metrics. Grep
the logs. Trace the requests. Inventory the resources. Every time. The steps are
known. The judgment about what matters is not.

The easy version is to ask an AI agent to investigate an outage. It will do
something. Without structure it misses steps. It hallucinates connections. It
drowns you in irrelevant data. It is easy to start. It is not simple to get
right.

## The structure

The toolkit is called `@webframp/aws-ops`. It depends on nine extensions, each
responsible for one domain:

- `aws/logs`: CloudWatch log queries
- `aws/metrics`: metric retrieval and anomaly detection
- `aws/alarms`: alarm state and history
- `aws/traces`: X-Ray service graphs and error traces
- `aws/inventory`: EC2 and Lambda resource inventory
- `aws/networking`: load balancers and NAT gateways
- `aws/alarm-investigation`: deep-dive alarm analysis
- `aws/cost-explorer`: cost data
- `github`: repository context

Each extension defines a model. Each model does one thing. You create instances of
them pointed at your region and your account. The ops extension composes them
into a workflow that runs the investigation in parallel where possible.

```bash
swamp model create @webframp/aws/logs aws-logs --global-arg region=us-east-1
swamp model create @webframp/aws/alarms aws-alarms --global-arg region=us-east-1
swamp workflow run @webframp/investigate-outage
```

Three commands. The workflow handles the rest.

## What the workflow does

The investigate-outage workflow executes seven steps:

1. Gather alarm summary and active alarms.
2. Analyze Lambda duration and errors. Analyze ELB 5XX rates and latency.
3. Retrieve the X-Ray service dependency graph and error traces.
4. List CloudWatch log groups and search for error patterns.
5. Inventory EC2 instances and Lambda functions.
6. List load balancers and NAT gateways with health status.
7. Generate an incident report that summarizes all findings.

Steps one through six run in parallel. Step seven waits for all of them to
finish. The report has six sections: alarms, metrics, traces, inventory,
networking, recommendations.

This is not a chatbot answering a question. It is a system executing a plan.

## Why nine extensions and not one

The first version was one extension. It did everything. It was easy to build.
It was not simple to understand, test, or extend.

When the alarm analysis needed to change, the entire extension had to change
with it. When I wanted to add cost data, I had to touch code that also handled
traces. The boundaries were wrong.

Nine extensions is more work to set up. Each one has its own manifest, its own
model definition, its own tests. But each one does exactly one thing. When the
alarm investigation logic improved, only `aws/alarm-investigation` changed.
When I added networking support, nothing else moved.

This is the Hickey distinction in practice. Nine extensions is harder to start.
It is simpler to maintain. Fewer parts per unit. Clearer relationships between
them.

## What I learned

The agent is good at execution. It is not good at deciding what to execute. The
first time I ran it without the workflow, it checked three alarms and declared
the system healthy. It missed a cold-start spike that was adding latency. The
data was there. The agent did not know to look.

The workflow is where that judgment lives. I decided the steps. I decided the
order. I decided what data to gather and how to present it. The agent runs the
steps. The report structure ensures nothing gets lost.

This is living in the system every day. I did not write most of
the code in these extensions. The agent did. But I wrote every workflow. I
defined every model boundary. I reviewed every report template. The
architectural decisions are mine. The implementation is not.

That division is the point. The agent made the work easy. The architecture is
what makes it simple.

## The machine that builds the machine

The toolkit is not the product. It is the machine that investigates my
infrastructure. It helps me decide faster. I built the machine. The machine
builds the understanding.

If I had let the agent design the system, it would have built one large
extension that did everything. It would have worked. It would not have been
simple. And the first time I needed to change it, I would have been lost.

The work is not the code. The work is the system that produces it.

[^1]: [The Architect's Instinct](/posts/architects-instinct/)
