---
layout: post
title: cross_sign_name_constraint_tool v0.0.2 and tlsrestrict_nss_tool v0.0.2 Released
author: Jeremy Rand
tags: [Releases, ncdns Releases]
---

We've released `cross_sign_name_constraint_tool` v0.0.2 and `tlsrestrict_nss_tool` v0.0.2.  These implement the functionality described in my previous post on [Integrating Cross-Signing with Name Constraints into NSS]({{site.baseurl}}2018/03/26/integrating-cross-signing-name-constraints-nss.html) (and the earlier posts that that post links to).

With this release, in theory Namecoin TLS negative overrides are supported in anything that uses NSS's trust store (including Firefox on all OS's, and Chromium on GNU/Linux, without requiring HPKP).

As usual, you can download it at the [Beta Downloads page]({{site.baseurl}}download/betas/#cross_sign_name_constraint_tool).

This work was funded by NLnet Foundation's Internet Hardening Fund.
