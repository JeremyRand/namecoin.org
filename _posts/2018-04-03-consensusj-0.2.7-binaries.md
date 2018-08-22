---
layout: post
title: "ConsensusJ-Namecoin v0.2.7 Binaries Available"
author: Jeremy Rand
tags: [Releases, libdohj Releases]
---

Binaries of ConsensusJ-Namecoin (the Namecoin lightweight SPV lookup client) v0.2.7 are now released on the [Beta Downloads page]({{site.baseurl}}download/betas/#consensusj-namecoin) page.  This is based on the source code that was [released earlier]({{site.baseurl}}2017/11/30/spv-lookup-0.2.7-beta-1.html).  Notable new things in this release:

* `leveldbtxcache` mode is merged to upstream libdohj, and has therefore had the benefit of more peer review.
* `leveldbtxcache` mode now only stores the name scriptPubKey, not the entire transaction.  This significantly improves syncup time and storage usage.  (Currently, it uses around 65 MB of storage, which includes both the name database and the block headers.)
* Many dependency version bumps.

**As usual, ConsensusJ-Namecoin is experimental.  Namecoin Core is still substantially more secure against most threat models.**
