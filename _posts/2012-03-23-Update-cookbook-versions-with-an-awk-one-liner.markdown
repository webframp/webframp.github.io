---
layout: post
title: "Update cookbook_versions with an awk one-liner"
date: 2012-03-23 13:18
comments: true
categories: shell
---

Sometimes it's the simplest things that remind me why I love the classic unix tools. Here's a quick way to fill in cookbook_versions for a chef environment using awk.

{% gist 2174632 %}

Of course, this is a ridiculously simple usage of awk. There's plenty [more](http://www.softpanorama.org/Tools/Awk/awk_one_liners.shtml) that [can](http://awk.info/?OneLiners) be [done](http://www.pement.org/awk/awk1line.txt) with just [a single line of awk](http://www.catonmat.net/blog/awk-one-liners-explained-part-one/).