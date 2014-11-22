---
layout: post
title: "Opscode Community Summit - Day Two"
date: 2011-11-30 20:41
comments: false
categories: opscode chef
---

The first historic Opscode Community Summit is officially over. See
yesterdays post for a basic description of the event and structure.

After another brief introduction, topics were discussed, a schedule
arranged and the agenda was live. Topics for day two included:

>  Managed Nodes a.k.a external entities
>  Chef for big data projects, such as hadoop, cassandra and similar
>  Ticket/triage process
>  Network monitoring, it sucks
>  Feature roadmaps, both short and long term


There's more, but I lost track due to poor note taking on my part.

By far the biggest announcement to come out of today was the fact that
Hosted Chef is moving from Ruby+NoSQL to Erlang+MySQL. For Opscode it
boils down to using the right tools for the job and after a hard look
at how Hosted Chef is being used it's clear this change will be for
the better. Erlang brings a ton of cool benefits and I look forward to
hearing about their migration path from the current platform. Lots of
nice performance graphs accompanied this discussion, led by Chris
Brown and Kevin Smith from Opscode.

[Sean Porter](http://twitter.com/portertech) led a discussion about monitoring and the how the
community is using the myriad of tools available to us. Basically
monitoring is hard and has subtle challenges at scale in current
implementations. He showed off [sensu](http://portertech.ca/2011/11/01/sensu-a-monitoring-framework/) and discussed ways for
possible collaboration with the community. I hope to dig in to this a
little more since I've been thinking about problems in around
monitoring, metrics and alerting for a little while lately.

![Day One Agenda](http://farm7.staticflickr.com/6039/6431812797_4cc2d00eba.jpg "Agenda Board - Day 2")

Overall what I most enjoyed about the summit is the sense of community
that is clearly maturing around chef. With so many passionate people
and hard problems, there's bound to be some neat things on the
horizon. It'll be exciting to see what the next year holds in store
for Opscode and the chef community.

Check out further details on the individual sessions on the [Opscode wiki](http://wiki.opscode.com/display/chef/Day+2)
