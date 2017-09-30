---
layout: post
title: "Namecoin TLS for Firefox: Phase 3 (Latency Debugging)"
author: Jeremy Rand
tags: [News]
---

I [recently mentioned]({{site.baseurl}}2017/09/30/firefox-tls-webext.html) performance issues that I observed with the Firefox TLS WebExtensions Experiment.  I'm happy to report that those performance issues appear to have been a false alarm, due to 2 main reasons:

1. I initially observed the performance issues in a Fedora VM inside Qubes.  I decided on a hunch to try the same code on a Fedora live ISO running on bare metal, and the worst-case latency decreased from ~20 ms to ~6 ms.  The spread also decreased a lot.  I can think of lots of reasons why Qubes might have caused performance issues, e.g. the use of CPU paravirtualization, the I/O overhead associated with Xen, the limit on logical CPU cores inside the VM, the use of memory ballooning, competition from other VM's for memory and CPU, and probably lots of other reasons.  In any event, my opinion is that the fault here lies in Qubes, not my code.
2. The Firefox JavaScript JIT seems to improve performance of the WebExtensions Experiment and of the WebExtension each time it runs.  After running the same code (on bare-metal Fedora) against 6 different certificates, the latency decreased from ~6 ms to ~1 ms, and it was still monotonically decreasing at the 6th cert.  Testing this code with many repeats is tricky, because it caches validation results per host+certificate pair, and I don't have a large supply of invalid certificates to test with.

The happy side effect of this debugging is that my current code is now quite a lot closer to Mozilla's standard usage patterns, since I spent a lot of time trying to figure out whether something that I was deviating from Mozilla on was responsible for the latency.  Huge thanks to Andrew Swan from Mozilla for providing lots of useful tips in this area.

I believe that there are still several optimizations that could be made to this code, but for now I'm reasonably satisfied with the performance.  (Whether Mozilla will want further optimization is unclear; I'll ask them later.)  My next step will be to set up negative overrides in the WebExtensions Experiment and the WebExtension.  After that, I'll be looking into actually making the WebExtension ask Namecoin for the cert data (instead of returning dummy data).  Then comes code cleanup, code optimizations, and a patch submission to Mozilla.

This work was funded by NLnet Foundation's Internet Hardening Fund.
