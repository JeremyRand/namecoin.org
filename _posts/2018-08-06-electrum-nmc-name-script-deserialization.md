---
layout: post
title: "Electrum-NMC: Name Script Deserialization"
author: Jeremy Rand
tags: [News]
---

I previously wrote about [making ElectrumX (the server) handle name scripts]({{site.baseurl}}2018/07/15/electrumx-name-scripts.html).  Now that that's out of the way, the next step is making Electrum-NMC (the client) handle name scripts as well.  I now have Electrum-NMC deserializing name scripts.

Most of the details of this work are fairly mundane implementation details that probably won't interest most readers.  So, instead, how about a screenshot?

![A screenshot of name transactions visible in the Electrum-NMC History tab.]({{site.baseurl}}images/screenshots/electrum-nmc/2018-07-17-Names-in-History-Tab.png)

This work was funded by NLnet Foundation's Internet Hardening Fund.
