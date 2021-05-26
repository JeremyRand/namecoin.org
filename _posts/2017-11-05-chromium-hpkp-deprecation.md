---
layout: post
title: "What Chromium's Deprecation of HPKP Means for Namecoin"
author: Jeremy Rand
tags: [News]
---

Readers who've been paying attention to the TLS scene are likely aware that [Google has recently announced that Chromium is deprecating HPKP](https://scotthelme.co.uk/the-death-knell-for-hpkp/).  This is not a huge surprise to people who've been paying attention; HPKP has had virtually no meaningful implementation by websites, and many security experts have been warning that HPKP is too dangerous for most deployments due to the risk that websites who use it could, with a single mistake, accidentally DoS themselves for months.  The increased publicity of the RansomPKP attack drove home the point that this kind of DoS could even happen to websites who *don't* use HPKP.  I won't comment on the merits of HPKP for its intended purpose.  However, readers familiar with Namecoin will probably be aware that Namecoin's TLS support for Chromium relies on HPKP.  So, what does HPKP's deprecation mean for Namecoin?

First off, nothing will happen on this front until Chrome 67, which is projected to release as Stable on May 29, 2018.  (Users of more cutting-edge releases of Chromium-based browsers will lose HPKP earlier.)  When Chrome 67 is released, I expect that the following behavior will be observed for the current ncdns release (v0.0.5):

* The ncdns Windows installer will probably continue to detect Chromium installations (because the HPKP state shares a database with the HSTS state, which isn't going anywhere).  The NUMS HPKP installation will appear to succeed.
* ncdns will continue to be able to add certificates to the trust store.  This means that `.bit` websites that use TLS will continue to load without errors.
* The NUMS HPKP pin will silently stop having any effect.  This means that Namecoin's TLS security will degrade to that of the CA system.  A malicious CA that is trusted by Windows will be able to issue certificates for `.bit` websites, and Chromium will accept these certificates as valid even when they don't match the blockchain.

Astute readers will note that this is the 4th instance of a browser update breaking Namecoin TLS in some way.  (The previous 3 cases were Firefox breaking Convergence, Firefox breaking nczilla, and Firefox breaking XPCOM.)  In this case, we're reasonably well-prepared.  Unlike the Convergence breakage (which Mozilla considered to be a routine binary-size optimization) and the nczilla breakage (which Mozilla considered to be a security patch), HPKP is a sufficiently non-niche functionality that we're finding out well in advance (much like the XPCOM deprecation).  As part of my routine work at Namecoin, I make a habit of studying how TLS certificate verification works in various common implementations, and regularly make notes on ways we could hack them in the future to use Namecoin.  Based on a cursory review of my notes, there are at least 7 possible alternatives to Chromium's HPKP support that I could investigate for the purpose of restoring Namecoin's TLS security protections in Chromium.  3 of them would be likely to qualify for NLnet funding, if we decided to divert funding from currently-planned NLnet-funded tasks.  (It's not clear to me whether we actually will divert any funding at this point, but we do have the flexibility to do so if it's needed.)  None of those 7 alternative approaches are quite as nice as our NUMS HPKP approach (which is why we've been using NUMS HPKP up until now), but such is life.

In conclusion, while this news does highlight the maintenance benefits of using officially approved API's rather than hacks (which, it should be noted, is my current approach for Firefox), at this time there is no reason for Namecoin to drop TLS support for Chromium.  I'm continuing to evaluate what our best options are, and I'll report back when I have more information.

This work was funded by NLnet Foundation's Internet Hardening Fund.
