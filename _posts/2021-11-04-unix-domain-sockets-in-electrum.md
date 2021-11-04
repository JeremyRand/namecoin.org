---
layout: post
title: "Unix domain sockets in Electrum"
author: yanmaani
tags: [News]
---

I recently sent in a patch to Electrum which adds support for Unix domain sockets. After review by ghost43 and Jeremy Rand, who is also a Namecoin developer, this patch has been merged. As a result, Electrum now has support for Unix domain sockets.

It is envisioned that this will be useful for `ncdns`, our DNS resolver, because it will allow Electrum-NMC to provide an RPC interface to `ncdns` without having to bind to a network port. This reduces attack surface, making the application more secure. The functionality can also be used by other Electrum users for other use-cases, and some are even suggesting to make it the default on Unix platforms.

I would like to thank Jeremy Rand and ghost43 for reviewing this pull request, pointing out several important details I missed and giving me helpful feedback on future-proofing and codebase standards.

This work was funded by NLnet Foundation's NGI0 Discovery Fund.
