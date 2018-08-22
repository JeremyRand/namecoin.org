---
layout: post
title: "Namecoin TLS for Firefox: Phase 5 (Moving the Override Cache to C++)"
author: Jeremy Rand
tags: [News]
---

In [Phase 4]({{site.baseurl}}2017/10/11/firefox-tls-threads.html) of Namecoin TLS for Firefox, I mentioned that more optimization work remained (despite the significant latency improvements I discussed in that post).  Optimization work has continued, and I've now moved the override cache from JavaScript to C++, with rather dramatic latency improvement as a result.

Prior to this optimization, my C++ code would synchronously call the WebExtensions Experiment to retrieve override decisions, and the Experiment would block until the WebExtension had returned an override decision.  At this point the decision would be added to the cache within the Experiment, and then the C++ code's call to the Experiment would return.  I had long suspected that this was a major latency bottleneck, for 2 reasons:

1. JavaScript generally is inefficient.
2. After Firefox's built-in certificate verification completed, control had to flow from C++ to JavaScript (which adds some latency) and then from JavaScript back to C++ (which adds latency as well).

With my latest changes, the control flow changes a lot:

1. The synchronous API for the C++ code to get positive override decisions from the Experiment is removed.
2. The override decision cache in the Experiment is removed.
3. An override decision cache is added to the C++ code.
4. An API is added to the C++ code for the Experiment to notify when an override decision has just been received from the WebExtension.  This API adds the decision to the C++ override decision cache.
5. The C++ code that gets a positive override now simply blocks until an override decision has appeared in the cache; it doesn't make any calls to JavaScript of any kind.

The advantages of this are:

1. A lot of inefficient JavaScript code is removed from the latency-critical code paths, in favor of more efficient C++.
2. Control never flows from C++ to JavaScript in order to retrieve the override decision (saves latency), and the flow from JavaScript to C++ can occur in parallel with Firefox's built-in certificate verification (saves latency as well).

I was originally hoping to use a thread-safe data structure for the C++ override decision cache, and noticed that Mozilla's wiki mentioned such a data structure.  However, I couldn't actually find that data structure in Mozilla's source code.  After a few hours of grepping and no luck figuring out what the wiki was referring to, I asked on Mozilla's IRC, and was told that the wiki was out of date and that the thread-safety features of that data structure were long ago removed.  So, the cache is only accessible from the main thread, and cross-thread C++ calls will still be needed to access it from outside the main thread.  This isn't really a disaster; cross-thread C++ calls aren't massively bad.

Since I wrote up some really nice scripts for measuring latency for Phase 4, I reused them for Phase 5 to see how things have improved.

<img src="{{site.baseurl}}data/webextensions-latency/2017-10-19/raw-data_html_e7e6307c38d1bb1a.png">

<img src="{{site.baseurl}}data/webextensions-latency/2017-10-19/raw-data_html_4abcef16d0d9da8.png">

This is a quite drastic speedup.  The gradual speedup over time has vanished, which suggests that I was right about it being attributable to the JavaScript JIT warming up.  (However, it should be noted that this time I did a single batch of 45 certificate verifications, so this may be an artifact of that change too.)  More importantly, based on the fact that uncached and cached overrides are indistinguishable in the vast majority of cases, it can be inferred that the Experiment's decision usually enters the C++ code's decision cache before Firefox's built-in certificate verification even finishes.  (The occasional spikes in uncached latency seem to correspond to cases where that's false.)

The raw data is available [in OpenDocument spreadsheet format]({{site.baseurl}}data/webextensions-latency/2017-10-19/raw-data.ods) or [in HTML format]({{site.baseurl}}data/webextensions-latency/2017-10-19/raw-data.html) as before.  The median uncached latency for positive overrides has decreased from 375 microseconds in Phase 4 to 29 microseconds in Phase 5.

It should be noted that negative overrides haven't yet been converted to use the C++ override decision cache.  I expect them to be slightly slower than these figures, because negative overrides will have 1 extra cross-thread C++ call.

The same disclaimer as before applies: this data is not intended to be scientifically reproducible; there are likely to be differences between setups that could impact the latency significantly, and I made no effort to control for or document such differences.  That said, it's likely to be a useful indicator of how well we're doing.

At this point, I am fully satisfied with the performance that I'm getting in these tests of positive overrides.  Converting negative overrides to work similarly is expected to be easy.  Of course, performance will probably be noticeably worse once the WebExtension is calling ncdns, so there's a good chance that after ncdns is integrated with the WebExtension, I'll be coming back to optimization.

For the short-term though, I'll be focusing on integrating the WebExtension with ncdns.

This work was funded by NLnet Foundation's Internet Hardening Fund.
