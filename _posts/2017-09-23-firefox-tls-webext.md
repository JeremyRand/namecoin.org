---
layout: post
title: "Namecoin TLS for Firefox: Phase 2 (Overrides in a WebExtension)"
author: Jeremy Rand
tags: [News]
---

As I [mentioned earlier]({{site.baseurl}}2017/09/24/firefox-tls-cpp.html), I've been hacking on a fork of Firefox that exposes an API for positive and negative certificate verification overrides.  When I last posted, I had gotten this working from the C++ end (assuming that a highly hacky and unclean piece of code counts as "working").  I've now created a WebExtensions Experiment that exposes the positive override portion of this API to WebExtensions.  (Negative overrides are likely to be basically identical in code structure, I just haven't gotten to it yet.)

But wait, what's a WebExtensions Experiment?  As you may know, Mozilla deprecated the extension model that's been used since Firefox was created, in favor of a new model called WebExtensions, which is more cleanly segregated from the internals of Firefox.  This has some advantages: it means that WebExtensions can have a permission model rather than being fully trusted with the Firefox internals, and it also means that Firefox internals can change without requiring WebExtensions to adapt.  However, it also means that WebExtensions are significantly more limited in what they can do compared to old-style Firefox extensions.  WebExtensions Experiments are a bridge between the Firefox internals and WebExtensions.  WebExtensions Experiments are old-style Firefox extensions that happen to expose a WebExtensions API.  WebExtensions Experiments have all the low-level access that old-style Firefox extensions had; among other things, this means I can access my C++ API from a WebExtensions Experiment written in JavaScript, and expose an API to WebExtensions that allows them to indirectly access my C++ API.

Creating the WebExtensions Experiment was relatively straightforward, given my prior experience with nczilla (remember, a WebExtensions Experiment is mostly just a standard old-style Firefox extension, like nczilla was).  I also created a proof-of-concept WebExtension that uses this API to make "Untrusted" errors be ignored.  While I was doing this, I also kept track of performance.  And therein lies the current problem.  When the Experiment simply returns an override without asking the WebExtension, the added overhead is generally 2 ms in the worst case (and often it's much less than 1 ms).  Unfortunately, Experiments and WebExtensions run in separate threads, and the thread synchronization required to get them to communicate increases the overhead by an order of magnitude: the worst-case overhead is around 20 ms (and it's very rarely less than 6 ms).

My assumption was that there was no way Mozilla would be merging this with that kind of performance issue; this assumption was confirmed by David Keeler from Mozilla.

On the bright side, it looks like there are several options for communicating between threads that should have lower latency (something like 1-2 ms overhead, assuming that documentation is accurate).  So I'll be investigating those in the next week or two.

As an interesting side note, Mozilla informs me that the current method I'm using to communicate between threads shouldn't be working at all.  I'm not sure why it's working when they say it shouldn't, but in any event it's probably a Very Bad Thing ™ to be using patterns that Mozilla doesn't expect to work at all, even without the latency issue (since such patterns might break in the future, which I certainly don't want).

This work was funded by NLnet Foundation's Internet Hardening Fund.
