---
title: "Six Million Events, Three Minutes of Trouble"
date: 2026-07-21
tags: ["swamp", "infrastructure", "trust"]
description: "A load test that briefly knocked out Swamp's telemetry pipeline, and what it took for that to read as curiosity instead of an attack."
---

I wanted to know if a swamp datastore could survive a swarm, so I asked an agent to design one, pointed it at ECS Fargate, and stepped away.

A custom Docker image, a fleet of remote workers, and a simple execution plan: model method calls firing events as fast as the workers could generate them. The final cluster ran 517 tasks: 500 loop workers each pushing telemetry at roughly 1.22 events per second, a single orchestrator running `swamp serve` against 50 models on a shard-first S3 datastore, and 16 serve-dispatch workers standing by for workflow batches. At full tilt that's north of 600 events a second, sustained. The point was lock contention: S3 versus my Postgres extension, under real concurrent write pressure, the kind a global team of hundreds would eventually produce whether I tested for it or not.

I wasn't watching when it climbed. Nick from Swamp caught it before I did and flagged it. I could have stopped it right there. I told him as much, and explained that I'd asked an agent to design the scenario and then stepped away from it. He told me not to bother. A few hundred workers were already enough to prove the point, and he'd rather evaluate the ceiling than cut it short.

So I kept going. Partway through, the ingest backlog had accumulated 6,275,351 pending events, 93 percent of the entire telemetry_events collection, live, while the swarm was still climbing. When he asked for a pause, I scaled the workers down and let them drain.

By the end I'd pushed roughly triple the platform's lifetime event volume through in a couple of hours. Outside of about three minutes of visible trouble, nobody would have known. I'd wanted to know if my datastores could take it, and maybe a little how he'd respond to someone stress-testing at that scale. Curiosity was the part he valued, not caution.

In most shops, containment planning assumes a stranger. You size the isolation boundary for the worst case, cap what any single actor can produce, and build the controls so nobody has to make a judgment call about who's on the other end while the system is under load. The controls exist because the judgment is too expensive to make in real time.

I asked an agent to design the scenario, pointed it at my own infrastructure, and wasn't watching when it climbed to 500 workers and started reporting telemetry to Swamp, which briefly overwhelmed their ingest pipeline. By the containment logic, that's a failure: no ceiling on what I could produce, no isolation between my test and their collection infrastructure, no one watching while it ran.

Nobody on the other end reached for that framing. He didn't ask why I hadn't capped it, or why I'd walked away from something that big. He asked me to pause, and then he told me it was the best kind of thing that could have happened. 

## What got evaluated

He was responding to the reason behind it, not the outage itself, and he graded it live, while the backlog was still climbing, not after the fact. I wasn't probing for a way in, and I was arguably careless in the most literal sense of the word, having handed the controls to an agent with an [intent that was precise on the question and silent on the ceiling](https://stack72.dev/intent-is-architecture/). I was running a legitimate capacity question at the only scale that would actually answer it. You cannot learn what breaks a system at 500 concurrent workers by imagining it. You have to send the events.

This is not how platforms usually behave, because it requires the platform to trust something it cannot verify from a metrics dashboard: the motive of the person on the other end of the load. Rate limits and containment policies exist precisely because you can't usually make that judgment fast enough, or trust it broadly enough, to let it substitute for a control.

## Accumulated proof

It worked because the evidence was already there before the swarm was. I'd been using Swamp at scale for months, writing extensions for the registry and building in public against real infrastructure. When 500 workers started flooding their telemetry pipeline, the people on the other end didn't need to guess whether the person responsible was invested in the platform's success. They had months of accumulated proof.

That context is not something you can establish after an incident. It exists before the incident or it doesn't exist at all. Take away that history and the same telemetry flood is indistinguishable from an attack, and the response to it looks completely different.

## The lesson

Containment planning optimizes for a stranger: assume nothing about who's on the other end, cap for the worst case, build the control so the judgment call never has to happen live. That's correct default behavior, and I'd tell any team of hundreds building on Swamp to run their own load exactly that way: announced, rate-limited, with someone's permission secured in writing before the first worker spins up.

But a system that treats every actor as a stranger has no way to be delighted by one. Nick's reaction wasn't naive. It was the payoff of already knowing who I was before the swarm hit. The containment failed. What happened after it failed is the part that changed.

I still owe them a heads-up next time: announced, capped, with someone watching the telemetry rate instead of stepping away. But I'm glad the uncapped, unattended version happened first. It's the only way I'd have found out that curiosity, not caution, was the thing he was measuring me on.
