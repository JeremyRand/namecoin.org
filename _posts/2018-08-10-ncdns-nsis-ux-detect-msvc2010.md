---
layout: post
title: "ncdns NSIS Installer UX: Detection of Visual C++ 2010 Redistributable Package"
author: Jeremy Rand
tags: [News]
---

If you've used ConsensusJ-Namecoin (our lightweight SPV name lookup client), you've probably noticed that we instruct users (on the Download page) to install the Microsoft Visual C++ 2010 Redistributable Package.  Failing to do this will result in the LevelDB library failing to load, which ends up causing some incredibly misleading error messages, after which `namecoinj-daemon` will terminate.  Unfortunately, in the Real World™, users don't reliably follow instructions.  (Confession: I've failed to follow this instruction when setting up test VM's before, and took quite a while to figure out what was broken.)

But how to improve the UX here?  Distributing the relevant package is legally questionable, so we can't do that.  However, an alternative is to make the ncdns NSIS installer (which handles user-friendly installation on Windows) detect whether the package is already installed, and display a more user-friendly error during installation so that users know what's wrong and how to fix it.

We already handle this concept to some extent.  For example, here's what the NSIS installer displays when Java isn't already installed:

!["Cannot use BitcoinJ SPV client (Java must be installed)"]({{site.baseurl}}images/screenshots/ncdns-nsis/spv-missing-java-2018-08-10.png)

Compare this to the typical display when Java *is* installed:

!["Install and use the BitcoinJ SPV client (lighter, less secure)"]({{site.baseurl}}images/screenshots/ncdns-nsis/select-spv-dnssec-trigger-2018-07-31.png)

So, over the last day or so, I hacked on the ncdns NSIS script to make it detect the Microsoft Visual C++ 2010 Redistributable Package (like we already do for Java), and refuse to install ConsensusJ-Namecoin if it's not found:

!["Cannot use BitcoinJ SPV client (Microsoft Visual C++ Redistributable Package must be installed)"]({{site.baseurl}}images/screenshots/ncdns-nsis/spv-missing-vc2010-2018-08-10.png)

This was an excellent excuse for me to get some more experience with NSIS.  Hopefully it saves some users from head-banging confusion.

(It should be noted, of course, that the "right way" to solve this is to actually make ConsensusJ-Namecoin not require any non-free Microsoft dependencies.  This is something we'll probably look into as we move towards reproducible builds.  That said, Windows users are already trusting non-free Microsoft code, so this isn't as big a deal as one might think.)

This work was funded by NLnet Foundation's Internet Hardening Fund.
