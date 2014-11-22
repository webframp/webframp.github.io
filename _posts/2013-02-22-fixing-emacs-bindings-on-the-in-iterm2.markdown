---
layout: post
title: "Fixing emacs bindings in iTerm2"
date: 2013-02-22 20:05
comments: true
categories: emacs
---

I spend a decent amount of time in org-mode, using Emacs + iTerm2 on a
Macbook Air. I fixed a recent annoyance I encountered when using Emacs
org-mode to track time for a project.

Org-mode has a handy
[time tracking feature](http://orgmode.org/org.html#The-clock-table)
and as the manual points out both `C-S-<up/down>` `S-M-<up/down>` can
be used to call various functions for adjusting recorded time. In
particular I wanted to use `org-timestamp-down` to add some time for a
CLOCK item I had forgotten to start before a Skype call.

Obviously though, I hadn't used `S-M-down` previously because when I
did instead of adjusting the time, I only got the raw escape code `[1;10B`.

I tried adjusting the settings in iTerm2 to no avail, so after a bit
of a meandering search down the emacs rabbit hole I found a solution
that worked well for me. 

```scheme
(define-key input-decode-map "\e[1;10A" [M-S-up])
(define-key input-decode-map "\e[1;10B" [M-S-down])
(define-key input-decode-map "\e[1;10C" [M-S-right])
(define-key input-decode-map "\e[1;10D" [M-S-left])

(define-key input-decode-map "\e[1;3A" [M-up])
(define-key input-decode-map "\e[1;3B" [M-down])
(define-key input-decode-map "\e[1;3C" [M-right])
(define-key input-decode-map "\e[1;3D" [M-left])
```

One handy tip in discovering these keybindings was to use `cat` to
display the raw keycodes:

```sh
$ cat
[1;10A
[1;10B
[1;10C
[1;10D
```

This can also be done use `C-q` in emacs if you prefer, both will give
you the results you need.

