---
layout: post
title: Registering Names with the Raw Transaction API
author: Jeremy Rand
tags: [News]
---

The refactorings to the raw transaction API that I [mentioned earlier]({{site.baseurl}}2017/09/13/raw-tx-api-refactoring.html) have been merged to Namecoin Core's master branch.  I've been doing some experiments with it, and I used it to successfully register a name on a regtest network with only one unlock of my wallet (which covered both the `name_new` and `name_firstupdate` operations).

I also coded support for Coin Control and Fee Control for name registrations, although this code is not yet tested (meaning that if Murphy has anything to say about it, it will need some fixes).

So far the code here is a Python script, so it still needs to be integrated into Namecoin-Qt.  Brandon expects this to be relatively straightforward once I hand off my Python code to him.  Major thanks to Daniel for getting the API refactorings into Namecoin Core, and thanks to Brandon for useful discussions on how best to structure the code so that it can be integrated into Namecoin-Qt smoothly.

Hopefully I'll have more news on this subject in the next couple weeks.

This work was funded by NLnet Foundation's Internet Hardening Fund.
