---
layout: post
title: "ncp11 Nightly Builds Now Available"
author: Jeremy Rand
tags: [News]
---

More work has been done with pkcs11mod and ncp11.

First off, a bunch of additional missing functions have beeen added to p11mod.  A lot of these functions came from Oureachy applicants, since I used pkcs11mod as a contribution task repo for Outreachy.  Some of these previously-missing functions may be useful to smartcard use cases, so if you've been holding off on touching p11mod for a smartcard project, it may be time to look again.

Next, I discussed the p11 API with Miek, specifically some shortcomings that make it hard for Namecoin to emulate the p11 API.  We've agreed that Namecoin will maintain a fork with API changes for about 6 months, and then I'll submit the changes upstream once I'm satisfied that it works in production.  The changes have a Concept ACK.  p11mod has beeen updated to use the fork.

The p11trustmod library is fully working now, although there are no integration tests for it yet.  But how did I test it, then?  The ncp11 codebase has also now been rewritten to use p11trustmod!  Currently it only does positive overrides, and I've only tested it with Firefox on GNU/Linux, but it works.  For reference, the old ncp11 was 1351 lines of code; the rewrite is only 228.  Not a bad result; seems the abstraction work done on p11mod and p11trustmod was well-worth it.  There is currently a quirk where ncp11 has to mark intermediate CA's as trusted; only marking the Namecoin root CA as trusted does not make the certs pass validation.  I intend to debug this later, but trusting intermediate CA's is harmless here.

For anyone who wants to play around with this, ncp11 Nightly builds are now available on the Beta Downloads page.  You'll also need a Nightly version of Encaya, as a bunch of bugfixes were needed on the Encaya side.  Installation instructions don't exist yet, but if you know how to install Encaya, and you know how to install PKCS#11 modules into Firefox (e.g. for smartcards), you should be able to figure it out.

This work was funded by NLnet Foundation's Internet Hardening Fund.
