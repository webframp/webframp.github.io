---
layout: post
title: "Email in emacs"
date: 2014-01-07 22:07
comments: true
categories: emacs email
---

I genuinely dislike email, yet it's a necessary part of working and
communicating these days. I suppose there is occasional value found in
the community of certain mailing lists, but generally the way email is
used today is simply more of a distraction or interruption.

A while back Steve Losh described how to setup Mutt
[the way he likes](http://stevelosh.com/blog/2012/10/the-homely-mutt/).
It was an interesting read because he was detailed, technical and
clearly prefers very customizable tools. I agree with him, and
although I've long since abandonded Mutt I was motivated to describe
my version of powerful, customizable, terminal based email management.

This isn't intended to say that what I use is inherently better than
the setup Steve described, it's simply what works for me. Much of the
setup is similar. I used Mutt for many years before giving
[Sup](http://sup.rubyforge.org/) a try, and recently settling on the
configuration I describe here. Sup is still actively developed and a
great standalone mail client.

I do use Gmail and while the Gmail web interface has a great set of
keybindings for most everything, they aren't customizable so it's a
whole new set of bindings to learn. That's not a terrible thing, but I
spend most of my day in a text editor where the bindings (and
subsequent muscle memory) I know are very different than those
available in the Gmail web interface.

Several times in the past I've searched for a fast, configurable way
to use emacs for accessing my email. For one reason or another the
[existing options](http://www.emacswiki.org/emacs/CategoryMail) were
lacking in my opinion and didn't fit my needs. However, a little over
2 years ago I stumbled across [Notmuch](http://notmuchmail.org/),
which is billed as a: {"fast, global search and tag based email reader
to use within your text editor or in a terminal"}. This sounded like
exactly what I was looking for.

It's essentially a minimal interface to a Xapian index of your mail so
powerful [searching is easy](http://notmuchmail.org/searching/), but
[not limited](http://xapian.org/docs/queryparser.html).

I also primarily use OS X so the basic design is very similar to what
Steve mentions in his
[Overview](http://stevelosh.com/blog/2012/10/the-homely-mutt/#overview),
the difference is that I've removed Mutt from the picture and
substitute the notmuch
[emacs integration](http://notmuchmail.org/emacstips/).

Offlineimap and msmtp setup is essentially the same and I have two
accounts configured, one personal and one for work both of which are
google accounts.

## Setup

If you use OS X, it's easily available using [homebrew](http://brew.sh/):

  `brew install notmuch`

And from emacs, assuming you have
[melpa](http://melpa.milkbox.net/#/notmuch) setup, notmuch is only an
`M-x package-install` away.

This gist has the settings I use  in my emacs `load-path` to setup custom
identities for my personal and work email accounts and configure how
notmuch works.

{% gist 8313021 %}

There's plenty of additional customization available depending on how
deep the rabbit hole you choose to go. This setup has worked well for
me up to this point and still allows me to use the gmail web interface
in a pinch.
