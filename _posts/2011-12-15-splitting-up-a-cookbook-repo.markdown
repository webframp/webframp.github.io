---
layout: post
title: "Splitting up a cookbook repo"
date: 2011-12-15 13:12
comments: true
categories: chef git
---

It seems in the chef community lately there's a growing trend for
cookbooks to be kept in separate repos, or even separate branches in a
single repo. I wanted to share the script I used to split out the
community-cookbooks repo for [Heavywater](https://github.com/heavywater/) 

I knew the general git commands I needed to use, but it did take a few
local trial runs to get it exactly as I wanted. To create the actual
repos on github I made use of the excellent
[hub](http://defunkt.io/hub/) script.

While there's definitely a more elegant way to do this, maybe you
can use this as a starting point should you need to do the same.

{% gist 1479720 %}
