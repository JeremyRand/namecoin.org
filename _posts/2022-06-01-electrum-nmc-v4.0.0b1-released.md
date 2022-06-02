---
layout: post
title: "Electrum-NMC v4.0.0b1 Released"
author: Jeremy Rand
tags: [Releases, Electrum Releases]
---

We've released Electrum-NMC v4.0.0b1.  This release includes important UX improvements to reduce the risk of accidentally letting names expire, both on the wallet side and a resolution mechanism called [semi-expiration]({{ "/2022/02/18/preventing-expiration-mishaps-with-semi-expiration.html" | relative_url }}) that stops resolving names before they are permanently lost.  Since semi-expiration affects resolution results, we therefore recommend that all users upgrade, even if you do not own any names yourself, so that you see the same resolution results as everyone else.  Here's what's new since v4.0.0b0:

* Simplify GUI for editing TLS records.  (Patch by Jeremy Rand.)
* Add documentation links to DNS Builder GUI.  (Patch by Jeremy Rand.)
* Add server from deafboy.  (Patch by Jeremy Rand.)
* Improve UNOList datetime formatting consistency.  (Patch by Jeremy Rand, review by Daniel Kraft.)
* Add `expires_time` field to `name_show`.  (Patch by Jeremy Rand, review by Daniel Kraft.)
* Implement semi-expiration.  (Reported by Cypherpunks, patch by Jeremy Rand, review by Arthur Edelstein, Cyberia Computer Club, Cypherpunks, Cyphrs, Daniel Kraft, Diego Salazar, Forest Johnson, s7r, Somewhat, and Yanmaani.)
* Allow passing raw commitment to `name_new`.  (Patch by Jeremy Rand.)
* Show exact expiration timestamps for expired names.  (Patch by Jeremy Rand.)
* Warn user in UNOList when names are expiring soon.  (Reported by Diego Salazar, patch by Jeremy Rand.)
* Fix some i18n bugs.  (Patch by Jeremy Rand, review by Somewhat, Yanmaani, and Daniel Kraft.)
* Add Namebrow.se explorer.  (Patch by Jeremy Rand.)
* Produce packager-friendly tarballs.  (Reported by Jeremy Rand, patches by Yanmaani and Jeremy Rand, review by Jeremy Rand and SomberNight.)
* Switch to detached OpenPGP signatures (to follow upstream).
* Various improvements from upstream Electrum.

As usual, you can download it at the [Beta Downloads page]({{ "/download/betas/#electrum-nmc" | relative_url }}).

This work was funded by NLnet Foundation's Internet Hardening Fund and NLnet Foundation's NGI0 Discovery Fund.
