---
layout: post
title: "Namecoin TLS for Firefox: Phase 4 (Fun with Threads)"
author: Jeremy Rand
tags: [News]
---

In [Phase 2]({{site.baseurl}}2017/09/30/firefox-tls-webext.html) of Namecoin TLS for Firefox, I mentioned that negative certificate verification overrides were expected to be near-identical in code structure to the positive overrides that I had implemented.  However, as is par for the course, Murphy's Law decided to rear its head (but Murphy has been defeated for now).

The specific issue I encountered is that while positive overrides are called from the main thread of Firefox, negative overrides are called from a thread dedicated to certificate verification.  WebExtensions Experiments always run on the main thread.  This means that the naive way of accessing an Experiment from the C++ function that would handle negative overrides causes Firefox to crash when it detects that the Experiment is being called from the wrong thread.

Luckily, I had recently gained some experience doing synchronous cross-thread calls (it's how the Experiment calls the WebExtension), and converting that approach from JavaScript to C++ wasn't incredibly difficult.  (The only irritating part was that the Mozilla wiki's C++ sample code for this hasn't been updated for years, and Mozilla's API has made a change since then that makes the sample code fail to compile.  It wasn't too hard to figure out what was broken, though.)

After doing this, I was able to get my WebExtensions Experiment to trigger negative certificate verification overrides.

Meanwhile, I talked with David Keeler from Mozilla some more about performance, and it became clear that some additional latency optimizations beyond [Phase 3]({{site.baseurl}}2017/10/07/firefox-tls-latency.html) were going to be highly desirable.  So, I started optimizing.

The biggest bottleneck in my codebase was that basically everything was synchronous.  That means that Firefox verifies the certificate according to its normal rules, and only then passes the certificate to my WebExtensions Experiment and has to wait for the Experiment to reply before Firefox can proceed.  Similarly, the WebExtensions Experiment has to wait for the WebExtension to reply before the Experiment can reply to Firefox.  This means 2 layers of cross-thread synchronous calls, one of which is entirely in JavaScript (and is therefore less efficient).

The natural solution is to try to make things asynchronous.  I decided to start with making the Experiment's communication with the WebExtension asynchronous.  This works by adding a new non-blocking function to the Experiment (called from C++), which simply notifies it that a new certificate has been seen.  This is called immediately after Firefox passes the certificate to its internal certificate verifier (and before Firefox's verification happens), which allows the Experiment and the WebExtension to work in parallel to Firefox's certificate verifier.  When the WebExtension concludes whether an override is warranted, it notifies the Experiment, which stores the result in a cache (right now this cache is a memory leak; periodically clearing old entries in the cache is on the to-do list).

Once Firefox has finished verifying the certificate, it asks the Experiment for the override decision, but now the Experiment is likely to already have the required data (or at least be a lot closer to having it).  The C++ to Experiment cross-thread call is still synchronous (for now), but the impact on overall latency is greatly reduced.

Unfortunately, at this point Murphy decided he wanted a rematch.  My code was consistently crashing Firefox sometime between the C++ code issuing a call to the Experiment and the Experiment receiving the call.  I guessed that this was a thread safety issue (Mozilla doesn't guarantee that the socket info or certificate objects are thread-safe).  And indeed, once I modified my C++ code to duplicate the relevant data rather than passing a pointer to a thread, this was fixed.  Murphy didn't go away without a fight though -- it looks like Mozilla's pointer objects also aren't thread-safe, so I needed to use a regular C++ pointer instead of Mozilla's smart pointers.  (For now, that means that my code has a small memory leak.  Obviously that will be fixed later.)

After doing all of the above, I decided to check performance.  On both my Qubes installation and my bare-metal Fedora live system, the latency from positive overrides is now greatly reduced.  I don't have exact scientific measurements (I was just skimming the quite verbose debug output), but it looks like on Fedora, the worst-case latency (as usual, probably due to incomplete JavaScript JIT warmup) was around 4 ms (down from 6 ms), and the latency after a few verifications (to warm up the JIT) was somewhere in the vicinity of 400 to 600 μs (down from about 1 ms).

There are 2 additional major optimizations that I intend to do (which aren't yet started):

1. Make the C++ to Experiment calls asynchronous.  This way, the C++ code doesn't need to issue a synchronous cross-thread call to retrieve the override data from the Experiment.
2. Add an extra asynchronous call that lets Firefox notify the Experiment and the WebExtension as soon as it knows that a TLS handshake is likely to occur soon for a given domain name.  In Namecoin's case, this gives the WebExtension a chance to ask ncdns for the correct certificate before Firefox even begins the TLS handshake.  That way, by the time the observed certificate gets passed to the WebExtension, it will be likely to already know how to verify it.

At this point I'm not 100% certain whether I'll choose to do more optimization next, or if I'll focus on hooking the WebExtension into ncdns.

This work was funded by NLnet Foundation's Internet Hardening Fund.
