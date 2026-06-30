---
title: "Drift Detection. No New Calls Required."
date: 2026-06-29T10:00:00-04:00
tags: [swamp, aws, drift, infrastructure, observation]
description: "Zero API calls. Still found the security group rule you added."
---

The drift-state extension makes zero AWS API calls. It reads data that other
models already produced, diffs it against a stored baseline, and writes the
result as typed versioned state. The "drift detector" is a composition function
over existing observations.

This matters because drift detection is usually sold as a feature of a specific
tool. CloudFormation has drift detection. Terraform has plan. AWS Config has
compliance evaluations. Each one instruments its own slice and reports on what it
manages. Nothing composes across them.

The alternative: treat drift as a query over time. Observe at T0. Observe again
at T1. Diff. The drift detector does not need to know how the observations were
produced or what API calls were involved. It needs two snapshots of the same
typed data.

## Five independent observers

The upstream models each observe one domain in isolation:

- **adopt** observes VPCs, subnets, gateways, route tables, security groups, RDS
  clusters. Pure discovery, no management intent.
- **inventory** scans resource explorer for EC2, Lambda, S3, DynamoDB, KMS, and
  everything else AWS Config tracks.
- **config-compliance** reads AWS Config rule evaluations and stores non-compliant
  resources with their rule context.
- **dns-observation** reads Route53 zones and records, cross-references targets
  against known infrastructure to find orphaned entries.
- **event-topology** maps EventBridge rules, SNS topics, SQS queues, and Lambda
  event source mappings into a directed graph.

None of these models references drift-state. None was built for drift
detection. Each one runs independently, stores its output as versioned data in
the swamp datastore, and moves on. The observation layer is not coupled to the
analysis layer.

## What composition looks like

The drift-state model defines a mapping from upstream model to normalizer:

```typescript
const SOURCES = {
  adopt: { specName: "discovery", defaultModelName: "aws-adopt" },
  inventory: { specName: "scan", defaultModelName: "aws-inventory" },
  terraform: { specName: "read_state", defaultModelName: "terraform" },
  config: { specName: "compliance", defaultModelName: "aws-config-compliance" },
  dns: { specName: "orphans", defaultModelName: "aws-dns-observation" },
  event_topology: { specName: "graph", defaultModelName: "aws-event-topology" },
};
```

Each source has a normalizer function that extracts a canonical resource
identifier, a resource type, and a snapshot. The normalizer is the only
source-specific code. Everything downstream operates on the normalized shape:

```typescript
interface NormalizedResource {
  canonicalId: string;
  resourceType: string;
  account?: string;
  region?: string;
  snapshot: Record<string, unknown>;
  source: string;
}
```

`set_baseline` reads all upstream data, normalizes it, and stores the result.
`compute_drift` reads all upstream data again, normalizes it, and diffs each
resource's current snapshot against the stored baseline snapshot. Resources that
match are `in_sync`. Resources that differ are `drifted` with the specific
changed attributes recorded. Resources in baseline but missing from current are
also `drifted`. Resources in current but absent from baseline are `unknown`.

The diff function is a recursive object comparison. It does not know whether the
object represents a security group or a Lambda configuration. It compares
values.

## Running it against real infrastructure

I deployed a CloudFormation stack in a sandbox account with intentionally simple
resources: a VPC, security group, Lambda function, SQS queues, an EventBridge
rule targeting both the Lambda and an SQS queue, an SNS topic with a subscription,
and a Route53 zone with a dangling CNAME pointing at a nonexistent instance.

Populated all five upstream sources. Set baseline. 1868 total resources observed
across the account (the test stack plus everything else already present). Ran
`compute_drift`. Everything showed `in_sync` except for
133 config-compliance resources with a known normalizer ordering issue. Adopt and
inventory had zero false positives against their own baselines.

Then introduced drift:

```bash
# Lambda timeout: 3s -> 30s
aws lambda update-function-configuration \
  --function-name drift-state-test-fn --timeout 30

# Security group: add ingress rule
aws ec2 authorize-security-group-ingress \
  --group-id $SG_ID \
  --protocol tcp --port 8080 --cidr 10.0.0.0/8

# EventBridge: add SQS target to existing rule
aws events put-targets --rule drift-state-test-rule \
  --targets "Id=sqs-target,Arn=arn:aws:sqs:..."
```

Re-ran the upstream sources. Re-ran `compute_drift`.

Adopt found the security group drift precisely: `ingressRuleCount: 0 -> 1`. The
normalizer stores that count as part of the snapshot. Baseline had 0. Current
has 1. That is the entire diff for that resource.

Inventory detected 105 drifted resources, including the Lambda timeout change
and a larger set of resources that crossed a pagination boundary between
runs. The boundary drift is real signal about observation granularity, not noise
to suppress.

Event-topology surfaced the new SQS target as 5 `unknown` resources rather than
`drifted`, because they did not exist at baseline time. This is a different kind
of signal: something appeared that was never declared. The event graph confirms
it — the test rule's out-degree went from 1 to 2.

## The velocity surface

`get_drift_velocity` breaks drift rate down by resource type:

```json
{
  "AWS::EC2::SecurityGroup": { "driftedCount": 2, "totalCount": 47, "driftRate": 0.04 },
  "AWS::Lambda::Function": { "driftedCount": 1, "totalCount": 5, "driftRate": 0.2 },
  "AWS::Events::Rule": { "driftedCount": 1, "totalCount": 11, "driftRate": 0.09 },
  "AWS::S3::Bucket": { "driftedCount": 46, "totalCount": 67, "driftRate": 0.69 }
}
```

The S3 bucket drift rate at 69% is noise from the inventory scan's pagination
boundary. But it tells you something real: if your observation granularity
produces false positives, your normalizer needs work. The velocity surface makes
normalizer quality visible. You can measure it.

## The contract between layers

A higher-order model that composes data from other models operates at a different
abstraction boundary than a model that calls an API directly. The boundary has
a shape that only becomes visible when you build against it.

The CEL expression layer and the JavaScript runtime expose different interfaces
to the same data. CEL has `findBySpec()`. The JavaScript runtime does not. CEL
accesses resource content via `resource.attributes.field`. The JavaScript
`readResource()` returns the content object directly, no wrapper. These are not
bugs. They are two interfaces designed for different consumers: workflow
expressions versus method implementations. A higher-order model lives in the
method layer but reasons about data the way a workflow does. It straddles the
boundary.

The practical consequence: every `readResource` call in drift-state was failing
silently because `.attributes.entries` evaluated to `undefined.entries`, threw
into a catch block, and the method proceeded with empty state. 1736 resources
all showed `unknown` status. The fix was mechanical. The lesson was structural:
when you compose across layers, verify the contract at each boundary before
trusting the output.

One more contract surprise: the instance name `latest` is reserved. Swamp uses
it internally for version management. Writing to it fails with a clear error,
but nothing warns you at authoring time.

Normalizer determinism matters more than normalizer completeness. The
config-compliance normalizer uses a composite key of resource ARN and config
rule name. The same IAM user appears under multiple rules, and which rule's
evaluation lands first varies between runs. This produces 133 false positives
per cycle. The adopt and inventory normalizers produce identical output from
identical input, so they generate zero. If your normalizer is not deterministic
given the same upstream data, your drift detector measures normalizer instability
rather than infrastructure change.

## What fell out naturally

Drift detection was never a feature that needed purpose-built instrumentation.
It fell out of the observation layer producing typed, versioned data. The only
new code is normalizers (source-specific) and a diff function (generic).
Everything else is composition: read existing data, compare across time, write
the result.

The same observation data supports compliance posture tracking, topology change
detection, and resource lifecycle queries without additional API calls. The
observation layer already did the work. The analysis layer is pure computation
over stored data.

This is what "you were never declaring state" looks like in practice. The
declaration was scaffolding for an agent that could not observe. The observation
layer removes the need for the declaration. Once you have observations stored as
typed versioned data, the analysis you can build over them is limited only by the
questions you think to ask.

The drift detector is one question. It will not be the last.
