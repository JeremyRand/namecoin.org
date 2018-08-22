---
layout: post
title: Electrum-NMC v3.2.2 Released
author: Jeremy Rand
tags: [Releases, Electrum Releases]
---

We've released Electrum-NMC v3.2.2.  Here's what's new:

* Trezor support.
* Support AuxPoW and timewarp hardforks.  (AuxPoW is still experimental, but it does successfully sync now.)
* Fix running the GNU/Linux release without installing first.
* Improvements from upstream Electrum.

Please note that due to a bugfix that was needed for Trezor support, any coins stored in wallets from older versions of Electrum-NMC will probably be inaccessible from this version.  We recommend that you empty your old Electrum-NMC wallets into Namecoin Core prior to installing this version, and then recreate your Electrum-NMC wallets after installing this version.

As usual, you can download it at the [Beta Downloads page]({{site.baseurl}}download/betas/#electrum-nmc).

This work was funded by Cyphrs.
