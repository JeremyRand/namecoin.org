---
layout: post
title: "Electrum-NMC: AuxPoW Verification"
author: Jeremy Rand
tags: [News]
---

Previously, I covered [AuxPoW deserialization in Electrum-NMC]({{site.baseurl}}2018/07/01/electrum-nmc-auxpow.html).  The next steps are on-disk serialization/deserialization, and verifying the deserialized AuxPoW.  These are now implemented.

It turned out that on-disk serialization/deserialization was a lot easier than anticipated, because Electrum only writes headers to disk *after* they've been verified, and it never reverifies them.  So it was sufficient to simply strip the AuxPoW portion of the headers prior to writing them to disk, and to tweak the header deserialization code so that it won't error when it encounters AuxPoW-enabled headers that don't have AuxPoW data (this is secure since even if such headers are successfully deserialized, they would never pass verification if verification were attempted).

Next up, AuxPoW verification!

AuxPoW verification code was already present in Electrum-DOGE, and it was relatively straightforward to port over to the current Electrum codebase.  In the process, however, I noticed that Electrum-DOGE has a dependency [on a library called `btcutils`](https://pypi.org/project/btcutils/).  I can't find that library's source code on GitHub, nor does the PyPI page indicate a Git repository, an issue tracker, or even a license.  At worst, this might indicate a GPL violation in Electrum-DOGE; at best, this indicates that the library has probably never been audited and is unwise to rely on.  As a result, I re-implemented from scratch the single function in that library that Electrum-DOGE was using.

After implementing AuxPoW verification in Electrum-NMC, I received a `bits` mismatch error during blockchain verification (some time after AuxPoW activated).  This was, of course, due to the "time warp" difficulty retargeting hardfork that Namecoin adopted long ago.  (Since I was the author of the time warp hardfork support in libdohj, fixing this in Electrum-NMC was quite straightforward.)

Next, I noticed that verification of AuxPoW headers was incredibly slow.  Some use of profiling tools revealed a number of bottlenecks (mostly relatively boring stuff like unneeded hex/binary conversions and unneeded copy operations), which I fixed.

At this point, I decided to do a full sync from scratch to see if it verified the entire blockchain.  Unfortunately, it failed at chunk 77, due to an error from the ElectrumX server indicating that the response exceeded the server's configured 5 MB limit.  I infer that there were a bunch of unusually large Bitcoin coinbase transactions around that point in Bitcoin's history.  Luckily, the operator of the public Namecoin ElectrumX server quickly replied to my email and raised the limit.  At this point, Electrum-NMC was able to fully sync.

In terms of performance, Electrum-NMC on my [Talos II](https://www.raptorcs.com/) is able to sync from scratch in about 6 minutes, without using any checkpoints (other than the genesis checkpoint).

Since Namecoin is a good neighbor, I'm maintaining an `auxpow` branch in the Electrum-NMC Git repository, which is identical to upstream Electrum except that AuxPoW support is added.  This may be useful to other AuxPoW-based cryptocurrencies who want a starting point for porting Electrum to their cryptocurrency.  (Daniel already does something similar for Namecoin Core.)

It should be noted that I haven't carefully audited the components inherited from Electrum-DOGE, so the AuxPoW support in Electrum-NMC should not be relied on in critical applications -- it would not be surprising if Electrum-DOGE's AuxPoW code is slightly consensus-incompatible with Namecoin Core.  Verifying that is on my to-do list.

The next step in Electrum-NMC is adding support for name scripts.  That will be covered in a future post.

This work was funded by Cyphrs.
