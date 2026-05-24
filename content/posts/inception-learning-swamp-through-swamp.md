---
title: "Inception: Learning Swamp Through Swamp"
date: 2026-05-24T20:00:00-04:00
tags: [swamp, infrastructure, devops, ai, education]
description: "A sandbox that teaches swamp, managed by swamp"
---

The [sandbox](https://github.com/webframp/swamp-sandbox) that teaches you swamp
is itself managed by swamp. A system whose
primitives are general enough to manage cloud infrastructure, editorial
workflows, and security scans can also manage the disposable learning environment
that introduces all of those patterns.

## The problem with demos

Most developer education follows a pattern: here is a tool, here is a contrived
example, here is a README explaining how the contrived example relates to your
real work. The gap between the tutorial and production is where most people give
up.

The gap exists because the example cannot be real infrastructure without real
consequences. You cannot teach Terraform by managing production VPCs. You cannot
teach Kubernetes by deploying to a shared cluster where mistakes cost money. So
the demo is a simplified model of reality, and the student must bridge the
abstraction gap themselves.

Swamp's primitives are domain-agnostic. A model that observes system information
inside a container and a model that discovers AWS resources across an account are
both TypeScript files with Zod schemas, methods that produce versioned data, and
outputs queryable via CEL. The domains differ. The authoring patterns are
identical. If the learning environment is itself managed by those patterns, the
student never encounters the abstraction gap. What they practice is what they
use.

## What the sandbox manages

Four [extension models](https://github.com/webframp/swamp-sandbox/tree/main/extensions/models)
handle the sandbox lifecycle. Each is a TypeScript file with Zod schemas, the
same authoring experience you would use for any domain:

**[coder-server](https://github.com/webframp/swamp-sandbox/blob/main/extensions/models/sandbox-coder-server/mod.ts).**
Observes the Docker Compose process. Its `status` method queries the Coder API,
captures the running version and container ID, stores it as typed data. The same pattern you would use to observe any containerized
service.

**[coder-template](https://github.com/webframp/swamp-sandbox/blob/main/extensions/models/sandbox-coder-template/mod.ts).**
Pushes workspace template versions. Its `push` method wraps the Coder CLI with
credentials resolved from a vault at runtime. The same
pattern you would use to deploy configuration to any system with an API.

**[coder-workspace](https://github.com/webframp/swamp-sandbox/blob/main/extensions/models/sandbox-coder-workspace/mod.ts).**
Provisions and observes workspace containers. Its `create` method passes template
parameters, polls for readiness, and stores the workspace state.

**[coder-task](https://github.com/webframp/swamp-sandbox/blob/main/extensions/models/sandbox-coder-task/mod.ts).**
Dispatches Claude Code prompts into running workspaces. Its `dispatch` method
creates Coder tasks and tracks their lifecycle.

These models are specific to Coder, but nothing about their structure is specific
to a learning context. They use the same patterns any production model would: Zod
schemas, typed outputs, vault-resolved credentials. A student who reads the
source code is reading real infrastructure automation, not a simplified version
of it.

## Vault integration as the first lesson

The credential problem teaches vault primitives by necessity. Claude Code needs
an API key or Bedrock token to run inside the workspace container. That
credential must get from the operator's environment into the workspace without
appearing in model definitions, execution reports, or git history.

The [Makefile](https://github.com/webframp/swamp-sandbox/blob/main/Makefile)
detects credentials from the shell environment or `~/.claude/settings.json`,
stores them in a local encrypted vault, and creates model instances with vault
expressions:

```yaml
globalArguments:
  claudeProvider: '${{ vault.get(sandbox-creds, CLAUDE_PROVIDER) }}'
  awsBearerTokenBedrock: '${{ vault.get(sandbox-creds, AWS_BEARER_TOKEN_BEDROCK) }}'
```

When a method executes, the expressions resolve to actual values. The method
summary report shows the expressions, not the secrets. The data store contains
no plaintext credentials. The student learns vault semantics because the system
requires them, not because a tutorial told them to practice vault operations.

The adoption pattern: start with the simplest backend that works (local
encrypted files), solve a real problem (credentials for workspace provisioning),
and discover the primitive's properties through use.

## CEL queries across the full stack

Once all four models have run, the student can query the entire sandbox state
with a single expression:

```bash
# run this against your own datastore
swamp data query 'modelType.startsWith("sandbox/") && isLatest == true' \
  --select '{"model": modelName, "spec": specName}' --json
```

This returns the latest observation from every model: server running, template
built, workspace healthy, task dispatched. All typed, versioned, and queryable
from a single expression without reading individual model outputs.

The query works because every model wrote typed data to the same datastore. The
student configured none of this; the primitives produced it by default. When
state is typed and shared, composition emerges from the data layer rather than
requiring explicit wiring.

## Two commands

```bash
make bootstrap    # From zero to running sandbox
make destroy      # Full teardown
```

Behind `bootstrap`: Docker Compose starts the server, the CLI authenticates,
credentials enter the vault, and swamp models push the template and provision the
workspace. Behind `destroy`: the workspace model's `delete` method runs, Docker
Compose stops, volumes are removed.

The student runs one command and gets a learning environment. Later,
`make status` shows them the model methods that produced it. Later still, they
read the extension source code and find the same patterns they have been
practicing inside the workspace.

Imitation, variation, then departure:

**Imitate.** Run `make bootstrap`. Run the examples. Observe the outputs.

**Vary.** Run `swamp model method run coder-workspace status` directly. Write a
CEL query. Read an extension model's source code.

**Depart.** Write your own model. Extend the sandbox. Manage something real with
the same patterns you learned on the disposable environment.

## What I discovered building it

Three findings from building the sandbox that apply to any swamp extension
authoring:

**Vault expressions require string schemas.** A `z.enum(["anthropic", "bedrock"])`
field rejects `${{ vault.get(sandbox-creds, CLAUDE_PROVIDER) }}` at model
creation time because the expression is a string, not an enum value. The fix:
use `z.string()` for any field whose value will come from a vault at runtime.
Schema validation happens after expression resolution during method execution.

**Reserved names exist in the data layer.** The instance name "latest" is reserved
internally for version resolution. A model that writes
`context.writeResource("state", "latest", data)` will fail. Name your instances
by their semantic role: "current" for state observations, "last" for the most
recently dispatched task.

**Something must exist before swamp can observe it.** In this sandbox, Docker
Compose is the provisioning layer: it starts the Coder server that swamp then
observes and manages. In production, that layer might be CloudFormation creating
a VPC, Terraform provisioning an RDS cluster, or a DigitalOcean droplet spinning
up via their API. The tool does not matter. The boundary does: whatever creates
the resource that must exist before observation can begin lives below swamp.
Everything above that boundary (configuration, state tracking, drift detection,
task dispatch) routes through typed models.

You cannot observe what does not exist yet. But a swamp method can create a
resource through an imperative API call and write typed versioned data in the
same execution. The provisioning step and the observation step are the same step, with no
declaration file between intent and creation, no separate observation pass after
the fact. The boundary collapses: every resource enters the system with
typed history from its first moment.

## The recursion resolves

A student who traces the data flow from vault to model to workspace has
encountered every primitive swamp offers: models, methods, typed resources,
vaults, CEL queries, and extension authoring. All exercised inside the system
that created the environment they are sitting in.
