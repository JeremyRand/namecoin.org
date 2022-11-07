---
layout: post
title: "Namecoin Core 22.0 Released"
author: Jeremy Rand
tags: [Releases, Namecoin Core Releases]
---

Namecoin Core 22.0 has been released on the [Downloads page]({{ "/download/#namecoin-core-client-stable-release" | relative_url }}).

Here's what's new since 0.21.0.1:

* RPC
    * Deterministic salts.  (Reported by s7r; Design by Ryan Castellucci and Jeremy Rand; Patch by Yanmaani; Review by Daniel Kraft.)
    * Transaction queue.  (Patch by Yanmaani; Review by Jeremy Rand and Daniel Kraft.)
* GUI
    * `name_list` Qt GUI.  (Patch by Jeremy Rand; Review by Brandon Roberts, Randy Waterhouse, and Daniel Kraft.)
    * `name_pending` Qt GUI.  (Patch by Jeremy Rand; Review by Daniel Kraft.)
    * Edit contents of model instead of resetting.  (Patch by Jeremy Rand; Review by Daniel Kraft.)
    * `name_update` Qt GUI.  (Patch by Jeremy Rand; Review by Daniel Kraft.)
    * Name renew Qt GUI.  (Patch by Jeremy Rand; Review by Daniel Kraft and Randy Waterhouse.)
    * Add Renew Names button.  (Patch by Jeremy Rand; Review by Daniel Kraft.)
    * Fix double lock when creating a new wallet.  (Patch by Jeremy Rand; Review by Daniel Kraft.)
    * Name decoration in Qt GUI.  (Patch by Jeremy Rand; Review by Daniel Kraft and Brandon Roberts.)
* Misc
    * Fix name operations in descriptor wallets.  (Reported by Jeremy Rand; Patch by Daniel Kraft; Review by Jeremy Rand.)
    * Rebrand Cirrus build directory.  (Patch by Jeremy Rand; Review by Daniel Kraft.)
    * Rebrand Guix.  (Patch by Jeremy Rand; Review by Daniel Kraft.)
    * Add mainnet DNS seed from Jonas Ostman.  (Patch by Jeremy Rand; Review by Daniel Kraft.)
    * Update hardcoded seeds.  (Patch by Jeremy Rand; Review by Daniel Kraft.)
    * Fix some AuxPoW CI errors.  (Reported by Jeremy Rand; Patch by Daniel Kraft.)
* Numerous improvements from upstream Bitcoin Core.
