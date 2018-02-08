---
layout: post
title: "Namecoin TLS for Firefox: Phase 6 (Negative Override Cache in C++, WebExtension Aggregation, Coordination with Mozilla, and SQLite as Default)"
author: Jeremy Rand
tags: [News]
---

In [Phase 5]({{site.baseurl}}2017/10/29/firefox-tls-moving-cache-cpp.html) of Namecoin TLS for Firefox, I discussed the performance benefits of moving the positive override cache from JavaScript to C++.  I've now implemented preliminary work on doing the same for negative overrides.

The code changes for negative overrides' C++ cache are analogous to those for positive overrides, so there's not much to cover in this post about those changes.  However, I did take the chance to refactor the API between the C++ code and the JavaScript code a bit.  Previously, only 1 WebExtension was able to perform overrides; if multiple WebExtensions registered for the API, whichever replied first would be the only one that had any effect.  Now, each WebExtension replies separately to the Experiment, and the Experiment passes those replies on to the C++ code.  The Experiment also notifies the C++ code when all of the WebExtensions have replied.  Although this does add some extra API overhead, it has the benefit of allowing an override to take place immediately if a single WebExtension has determined that it should take place, even if the other (irrelevant) WebExtensions are still evaluating the certificate.

Unfortunately, at this point I merged upstream changes from Mozilla into my Mercurial repository, only to find that there was now a compile error.  I'm still figuring out exactly why this compile error is happening.  It looks like it's unrelated to any of the files that my patch touches; I suspect that it's due to my general lack of competence with Mercurial (Mozilla's codebase is the first time I've used Mercurial) or my similar general lack of competence with Mozilla's build system.

So, until I actually get the code to build, I won't be able to do performance evaluations of these changes.  Hence why there are no pretty graphs in this post.

Meanwhile, I reached out to Mozilla to get some feedback on the general approach I was taking.  (I had previously discussed high-level details with Mozilla, but this time I provided a WIP code patch, so that it would be easier to evaluate whether I was doing anything with the code that would be problematic.)  This resulted in a discussion about what methods should be used to prevent malicious or buggy extensions from causing unexpected damage to user security.  This is definitely a legitimate concern: messing with certificate verification is dangerous when done improperly, and it's important that users understand what they're getting when they install a WebExtension that might put them at risk.  That discussion is still ongoing, and it's not clear yet what the consensus will arrive at.

In other Firefox-related news, Firefox 58 switched the default NSS database format from BerkeleyDB to SQLite.  This has the happy side effect that my experimental ncdns branch for positive certificate overrides for Chromium on GNU/Linux is likely to work for Firefox 58 and higher (on all OS's) without significant changes.  However, there are 3 caveats:

1. That ncdns branch relies on NSS's `certutil`, for which Mozilla doesn't distribute official binaries.  GNU/Linux distributions usually include `certutil` (e.g. in Fedora's `nss-tools` package), but it's not obvious to me what we'll do for Windows and macOS.  It's probably possible to reproducibly cross-compile `certutil` (most of the relevant code is already built reproducibly by Tor), but this would be extra work for us that I'm not sure we have the resources to handle.
2. `certutil` is slow in SQLite mode.  Very slow.  800ms latency per certificate, to be specific.  I did some initial debugging of this, and found that this is mostly due to NSS failing to properly batch SQLite writes into a single transaction.  I've filed a bug with Mozilla about this, and while Mozilla confirms that this is a bug that should be fixed, it's not clear when it will be fixed.
3. Tor Browser doesn't use a persistent NSS database by default, so even once Tor Browser is rebased onto a version of Firefox >=58, my experimental ncdns branch won't work for Tor Browser (unless Tor Browser's configuration is modified to deliberately enable a persistent NSS database, which will decrease Tor Browser's security guarantees).

For these reasons, it's still beneficial to continue pursuing the WebExtensions route.  However, `certutil` is certainly a decent temporary kludge while we wait for WebExtensions support to get merged, and is also a decent contingency plan if for some reason we can't get WebExtensions support to happen.

This work was funded by NLnet Foundation's Internet Hardening Fund.
