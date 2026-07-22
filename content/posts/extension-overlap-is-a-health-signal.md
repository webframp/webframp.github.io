---
title: "Extension Overlap Is a Health Signal"
date: 2026-07-22
tags: [swamp, community, extensions, ecosystem]
description: "Two extensions covering the same domain means the ecosystem is growing. That's not a governance problem."
---

On July 20, `@swamp/container-image` merged into the official swamp extensions
repository. `@webframp/container-image`, my extension covering the same domain,
had been on the registry for 38 days already. Both handle OCI image builds, pipe
passwords via stdin, branch on container runtime, and store digest and timing
metadata.

My first reaction was territorial. Someone rebuilt what I built, in the
privileged namespace, without referencing the prior work. But the impulse to
prevent overlap or require acknowledgment both lead to the same place: an
ecosystem that discourages people from building things. That is worse than any
amount of duplication.

## What overlap tells you

When two people independently build extensions for the same system, the domain is
validated. One container-image extension is a personal project, but two confirms
the community needs it modeled. My extension supports buildah and nerdctl. The
official one has input injection guards and a pre-flight validation method. Those
different choices would not exist if only one author worked in the space, and
users benefit from having both available.

A registry where publishing costs a manifest, a TypeScript file, and
`swamp extension push` will produce duplicate coverage in popular domains the
same way npm produced `got`, `axios`, `node-fetch`, `undici`, and the built-in
`fetch`. The only way to prevent overlap is to gatekeep publication or discourage
independent authorship, and the swamp ecosystem benefits from neither.

## The trust hierarchy

The registry auto-resolves `@swamp/*` extensions. Community collectives require
`swamp extension trust add`. This creates a discoverability advantage for
official extensions, and that's intentional as a security boundary. New users get
safe defaults without evaluating every publisher.

The trust model verifies publishers, not fitness for purpose. An extension that
supports four container runtimes serves a different audience than one focused on
safety hardening for two, and which collective published it tells you nothing
about which one matches your stack.

## What coexistence looks like

| Capability | `@webframp` | `@swamp` |
|---|---|---|
| Inspect | ✅ | ❌ |
| Run | ❌ | ✅ |
| Validate (pre-flight) | ❌ | ✅ |
| Buildah / nerdctl | ✅ | ❌ |
| Apple Containers | ❌ | ✅ |
| Input injection guards | ❌ | ✅ |

Neither is a superset. A team running podman and buildah on Linux wants mine,
a team on macOS using Apple Containers wants theirs, and a team that needs both
inspect and run has a reason to pull both or contribute the missing method
upstream to whichever one they prefer.

## What the ecosystem needs to communicate

I talked to Paul Stack about this. We agree: overlap is a positive signal for
community adoption. More people building means the authoring experience is good
enough that independent solutions emerge without coordination.

A community extension author who publishes in June and discovers an official
extension in the same domain in July needs to have known that was possible before
it happened.

Community authors should understand that their work has value independent of what
the official collective publishes, and that `@swamp/foo` appearing on the
registry does not deprecate `@community/foo`. The official collective benefits
from searching the registry before publishing because knowing what exists helps
write better documentation and surface alternatives. And `swamp extension search`
should return all collectives with enough metadata to compare on capability
rather than namespace.

The registry could surface overlap at publish time by noting when an extension's
labels match existing extensions in other collectives, helping authors discover
each other rather than compete in the dark.

## Where this goes

A registry with zero overlap has one author, and a registry where overlap causes
acrimony has unclear expectations. The swamp extension ecosystem crossed a
threshold where multiple people want to model the same domains, and the work now
is making sure everyone involved understands that this is what community growth
looks like.
