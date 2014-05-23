---
layout: post
title:  "Switching Chroots Shortcut"
category: post
tags: [crouton]
date:   2014-04-28 00:35:00
---

I suppose I might as well make a real post at this point, so here goes:

If you want to make switching between ChromeOS and Crouton easier, go to your chroot's /etc/crouton/xbindkeysrc.scm and replace the default keyboard shortcuts with (control alt Left) and (control alt Right). This will allow you to switch between them with your arrow keys.

This keyboard shortcut works seamlessly with vertically arranged workspaces, as the keyboard shortcuts for scrolling through those are CtrlAltUp and CtrlAltDown. Replace Left and Right with Up and Down if you prefer horizontal workspaces. I prefer vertical workspaces because my LeftRight arrow keys are much larger than my UpDown ones, and I switch between ChromeOS and my chroot much more frequently than I scroll through workspaces.

/realpost

Damn this prose is uninspiring. I guess that's technical writing for ya.
