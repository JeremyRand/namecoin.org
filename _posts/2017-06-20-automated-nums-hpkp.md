---
layout: post
title: Automated Nothing-Up-My-Sleeve HPKP Insertion for Chromium
author: Jeremy Rand
tags: [News]
---

As I mentioned in [my previous post]({{site.baseurl}}2017/06/15/how-were-doing-tls-for-chromium.html), we protect from compromised CA's by using a nothing-up-my-sleeve (NUMS) HPKP pin in Chromium.  Previously, it was necessary for the user to add this pin themselves in the Chromium UI.  This was really sketchy from both a security and UX point of view.  I have now submitted a PR to ncdns that will automatically add the necessary pin.

This is implemented as a standalone program that is intended to be run at install.  It works by parsing Chromium's `TransportSecurity` storage file (which is just JSON), adding an entry for `bit` that contains the NUMS key pin, and then saving the result as JSON back to `TransportSecurity`.

I expect this to work for most browsers based on Chromium (e.g. Chrome and Opera), on most OS's (e.g. GNU/Linux, Windows, and macOS), but so far I've only tested with Chrome on Windows.  I don't expect it to work with Electron-based applications such as Riot and Brave; Electron doesn't seem to follow the standard Chromium conventions on HPKP.  I haven't yet examined Electron to see if there's a way we can get it to work.

This isn't yet integrated with the NSIS installer; I'll be asking Hugo to take a look at doing the integration there.
