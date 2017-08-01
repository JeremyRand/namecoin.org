---
layout: post
title: ncdns CryptoAPI Certificate Injection Merged
author: Jeremy Rand
tags: [News]
---

After quite a bit of code cleanup, ncdns has merged my PR for CryptoAPI dehydrated certificate injection.  This means that the ncdns master branch can now be used for Namecoin-based TLS certificate validation in Chromium and Chrome on Windows.  Huge thanks to Hugo Landau for the code review.

Hugo is still doing some code cleanup of the automated NUMS HPKP injection before it gets merged to the NSIS installer, so for now the Chromium/Windows support requires following some manual installation instructions.  The good news is that Hugo has addressed nearly all of my review there, so I'm hoping we can get fully automated Chromium support merged to the NSIS installer very soon, at which point we'll do a release.  Awesome work by Hugo on this.

This work was funded by NLnet.

Meanwhile, Namecoin TLS development focus has started to shift toward Firefox TLS support.
