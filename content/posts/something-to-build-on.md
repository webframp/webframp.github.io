---
title: "Something to Build On"
date: 2026-11-10T00:00:00-05:00
tags: [swamp, ai, devops, promise-theory]
description: "Swamp gives people a way to turn what they know into behaviors other people can use."
---

In 2011, I [wrote about the first day of the Opscode Community
Summit](/posts/opscode-community-summit-day-one/). What stood out was the mix
of people. Network engineers, sysadmins, and developers were in the same room,
bringing different experience to problems they shared. People chose the
discussions where they could contribute or learn something. The arrangement
trusted them to decide where their time was useful.

Reading Swamp's [Behavior-Centric AI Design](https://swamp-club.com/behavior)
brought me back to that post. The article explains an architecture, but the
reason I care about it starts with the same belief: people who understand a
problem should have the means to do something about it. Swamp gives that belief
a practical form for working with AI.

The early DevOps movement had plenty of arguments about tools. Underneath them
was a more useful ambition. The people writing software and the people running
it needed to work together well enough to deliver something that survived
production. John Willis's [2010 account of
DevOps](https://www.chef.io/blog/what-devops-means-to-me) put culture,
automation, measurement, and sharing together. A deployment tool could help.
So could an honest conversation with the person carrying the pager.

What appealed to me then still does. Knowledge is scattered through an
organization. The person who understands a particular failure may be several
layers away from the person authorized to fix it. Useful work depends on
getting that knowledge into the decisions that matter.

With AI, someone who knows what an investigation needs can build the software
to conduct it with much less implementation work. A person who knows why a
report is misleading has a better chance of fixing how it is produced. Some
of these problems have never justified weeks of a development team's time.
They are still worth solving, especially to the people who encounter them
every day.

Swamp helps make the resulting work usable again. An agent translates intent
into behaviors with defined inputs, methods, and structured outputs. Workflows
compose those methods, and the data they produce remains available for later
use. An LLM can still supply judgment where the task calls for it. The
repeatable procedure has a home in code.

That work survives the next run. A decision about which evidence an
investigation must gather can become part of the workflow. The person who made
that decision can inspect and change its implementation. Other people can use
it without reconstructing the conversation that led to it.

I wrote observers for AWS resources, each recording what it found as versioned
data. Later I built a
[drift detector](/posts/drift-detection-as-composition/) that read those
observations, normalized them, and compared them with a baseline. The detector
made no AWS API calls of its own. The observers had already done that work.

I hadn't planned for drift detection when I wrote the first observer.

Another engineer could use those observations to examine a different problem.
They would need to understand what the data represents and whether it is fresh
enough for their purpose. They could then spend their effort on the question
they came to answer, with the collection work already done.

Mark Burgess's [promise theory](https://markburgess.org/promiseFAQ.html) helps
explain why this can work. Its starting point is autonomous
participants making commitments about their own behavior. Cooperation requires
both an offer and someone willing to use it. Trust develops through experience
of those commitments being kept. In this vocabulary, an agent can be a person
or a piece of software; it need not be an LLM.

In Swamp, an observer undertakes to collect a particular kind of evidence. A
consumer relies on that evidence for a particular purpose. The usefulness of
their relationship depends on whether those expectations agree. A schema states
the agreement, and runs show whether it holds.

The drift detector showed me where such an agreement can fail. A mistaken
assumption about the data interface left it reporting unknown resources. I
traced the failure to the boundary where I had misread the contract and
corrected it.

People doing the work should be able to change how it is done. Someone
investigating a fault can add the observation that was missing, and someone
reading a report can ask the question its author never anticipated. The tools
have to support that, and so does the organization.

People publish Swamp extensions from their own domains, and others build on
them. A useful extension carries
some of its author's knowledge: which observations matter, how an API behaves,
what a result means. As people use and improve it, that knowledge becomes easier
to apply elsewhere.

AI helps the author write an extension and helps the next person fit it to a
different task. Each reuse repays some of the original effort, and a job that
was impractical to build from scratch can become small enough to attempt.

An initial question can be precise enough to begin and still change when the
first observations arrive. I want tools that let me follow that change, and
with Swamp I can return to the code and data as my understanding improves.

An investigation that finds the fault has done its job. If improving it also
lets the next engineer catch the fault earlier, the work reaches past the
incident that prompted it.

I recognize the optimism from that room in 2011 and it honestly hard to believe
so much time has passed. People brought what they knew and expected to leave
better able to do their work. That hope is never more alive that today using
Swamp and accepting using inference for just the right stages of the work. With
an agent doing much of the implementation, I can build the observer I need today
and leave it for someone whose question I haven't thought of yet.
