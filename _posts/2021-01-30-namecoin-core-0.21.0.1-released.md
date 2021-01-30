---
layout: post
title: "Namecoin Core 0.21.0.1 Released"
author: Jeremy Rand
tags: [Releases, Namecoin Core Releases]
---

Namecoin Core 0.21.0.1 has been released on the [Downloads page]({{site.baseurl}}download/#namecoin-core-client-stable-release).

Here's what's new since 0.19.0.1:

* `name_show` for expired names will error by default.  (Reported by Jeremy Rand; Patch by Yanmaani; Reviewed by Daniel Kraft.)
* `name_show` can accept queries by SHA256d hash.  (Reported by Jeremy Rand; Patch by Daniel Kraft; Reviewed by Jeremy Rand.)
* `name_firstupdate` and `name_update` now consider the value to be optional.  (Reported by Jeremy Rand; Patch by Yanmaani; Reviewed by Daniel Kraft.)
* Added `namepsbt` RPC method.  (Reported by Jeremy Rand; Patch by Daniel Kraft.)
* Added new DNS seed for testnet.  (Patch by Yanmaani; Reviewed by Daniel Kraft.)
* Fix build for WSL.  (Patch by Chris Andrew; Reviewed by Daniel Kraft.)
* Fix crash on macOS 10.13+.  (Reported by Jip; Analysis by Jip, Daniel Kraft, Andy Colosimo, and Cassini; Patch by DeckerSU; Reviewed by Daniel Kraft.)
* Code quality improvements for regression tests.  (Patch by Daniel Kraft.)
* Numerous improvements from upstream Bitcoin Core.
