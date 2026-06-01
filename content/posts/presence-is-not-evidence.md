---
title: "Presence Is Not Evidence"
date: 2026-06-01T10:00:00-04:00
tags: [devops, ai, swamp, security, deming]
description: "Deming told us. We made the signature the proof and called it quality."
---

Someone approved your last deploy. Maybe it was a thumbs-up on a pull request.
Maybe it was a green check from a required reviewer who opened the diff, scrolled
for eight seconds, and clicked "Approve." Maybe it was your own merge after CI
passed. Regardless of the ceremony's size, the question is the same: what did
that approval actually verify?

Not what it was supposed to verify. What it did verify. What evidence exists that
a specific risk was evaluated, a specific judgment was exercised, a specific
context was held that the system could not hold on its own.

In most teams, the honest answer is: the approval verified that the process was
followed. Someone with the right permissions clicked the right button at the
right time. That is the entire basis of trust for what shipped.

## The identity assumption

Software delivery trusts outputs because of who produced them. Who approved the
PR. Who has merge access. Who carries the title that confers authority over a
given system. At scale this becomes a change advisory board; at startup scale it
is the senior engineer whose blessing you need before deploying on a Friday. The
mechanism differs. The assumption is identical: if the right person approved it,
the right thinking happened.

This is Frederick Taylor's factory floor, dressed in YAML. Authority vested in
position. Quality guaranteed by role. We have no mechanism to verify whether the
thinking actually occurred.

Deming tried to kill this idea for forty years. His argument was not
subtle: you cannot inspect quality into a product after the fact. Quality comes
from the system that produces the work, not from the person who reviews it at the
end.[^1] The Japanese manufacturers who listened to him in the 1950s dismantled
American industrial dominance in the 1980s. The American managers who did not
listen responded by adding more inspectors.

We are still adding inspectors. Required reviewers. CODEOWNERS files. Mandatory
CI checks that block merge but that no one reads the output of. Each one is an
inspector stationed at the end of a line, asked to verify quality that should
have been designed into the process from the start.

## What agents dissolve

When an agent writes the code, reviews the pull request, runs the security scan,
and deploys the artifact, every step can produce versioned, typed, queryable
evidence. The agent does not have a title. It does not attend a meeting. It
cannot lend its reputation to a change record. All it produces is data: what
ran, against what criteria, with what results, at what time.

This strips the costume off the approval ceremony. The question that was always
hiding becomes unavoidable: what did human approval actually verify? Can you
point to the specific judgment that the human exercised, the specific risk they
evaluated, the specific context they held that the system could not?

The answer is visible for the first time: the human's role was purely procedural.
Confirm the sequence. Check the boxes. Decide whether to slow down and look
harder, and nearly always choose not to, because the queue was long and the
meeting was short.

That is not judgment. That is ceremony.

## Where judgment belongs

Human judgment is irreplaceable for a specific class of decisions: system design,
boundary placement, failure mode anticipation, the recognition of which questions
a security model must ask. The architect's instinct, the seam recognition, the
knowledge of what will compose safely and what will not. These cannot be
delegated.

But they should never have lived in the approval queue. They belong in the
design of the system that executes. The security team's highest-value work is
defining the typed criteria that every scan evaluates against, the compliance
baselines that every deployment verifies, the threat model schemas that every
agent consumes as data. Reviewing scan results on Tuesday afternoon is maintenance;
designing what gets scanned and why is architecture.

In [Swamp](https://swamp.club), this becomes concrete. A security team publishes
an extension with typed models: vulnerability thresholds, compliance baselines,
policy rules as structured data. Every workflow that runs against the same
datastore queries that context automatically. The security team's expertise lives
in the model design, not in a meeting.

```bash
swamp data query 'modelName == "compliance-baseline" && isLatest == true' --json
```

The evaluation happens in parallel with implementation. No queue. No handoff. The
security thinking moved upstream to where it always should have been: in the
system's epistemic infrastructure, before any specific piece of work exists.

## Identity collapses, provenance remains

Every artifact traces to the process that produced it. Not who ran it, but what
ran, with what inputs, against what versioned criteria. Any claim the system
makes about quality can be independently confirmed by querying the data layer,
because the evidence lives in typed, immutable, queryable state rather than in
someone's memory of a meeting. Every execution writes its inputs, outputs, and
evaluation criteria as versioned data. The swamp datastore holds the complete
history. A CEL query produces the compliance report.

This shifts what "senior" means. The senior engineer's value is no longer being
the person whose judgment is trusted on faith. It is designing systems whose
outputs are trustworthy by construction. Seniority becomes legible in a way that
institutional authority never required it to be. You can see the quality of the
system someone designed. You could never see the quality of the judgment someone
claimed to exercise in an eight-second review.

## The political wedge

Swamp produces better paper trails than any approval ceremony, automatically,
from every execution. That is the political argument for adoption: your
compliance team gets richer evidence with less human effort. Frame it as "better
audit trails" and you get organizational buy-in.

But frame it small and you miss the point. The paper trail is a side effect of
something deeper: a system where trust is earned by evidence, not conferred by
title. The approval queue dissolves not because agents made it faster, but
because the system makes its function visible for what it always was: ceremony
substituting for engineering.

Deming again: "A bad system will beat a good person every time."[^2] The inverse
is also true. A good system, one that produces evidence at every step, will
outperform any arrangement of talented people connected by queues and handoffs.
The talent belongs in the system design, not in the inspection process.

## The open questions

Two tensions this argument cannot resolve:

What happens to the people whose authority rested on presence rather than
evidence? The required reviewer who never read the diff. The senior engineer
whose approval was pro forma. The security reviewer who checked the box without
running the scan. These roles existed because the system never
required them to demonstrate what they verified. When evidence becomes the
standard, presence stops counting. That is a career disruption for people who
built careers on institutional trust.

And the harder question: what happens to the next generation who will never
accumulate instinct through friction? The architect's judgment was forged by
years of slow, manual engagement with failing systems. When agents handle the
mechanical work from day one, where does that judgment come from? The friction
was painful and wasteful, but it was also how people learned. We have not yet
built the replacement for learning-through-difficulty. We have only removed the
difficulty.

These are not problems that a blog post resolves. They are the shape of what
comes next.

[^1]: W. Edwards Deming, *Out of the Crisis* (MIT Press, 1986). The core argument: quality is a property of the system, not a result of inspection.
[^2]: Attributed to Deming; the phrasing varies across sources but the principle is central to his work on systems thinking.
