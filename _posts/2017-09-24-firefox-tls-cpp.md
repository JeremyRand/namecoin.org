---
layout: post
title: "Namecoin TLS for Firefox: Phase 1 (Overrides in C++)"
author: Jeremy Rand
tags: [News]
---

Making TLS work for Namecoin websites in Firefox has been an interesting challenge.  On the positive side, Mozilla has historically exposed a number of API's that would be useful for this purpose, and we've actually produced a couple of Firefox extensions that used them: the well-known Convergence for Namecoin codebase (based on Convergence by Moxie Marlinspike and customized for Namecoin by me), and the much-lesser-known nczilla (written by Hugo Landau and me, with some code borrowed from Selenium).  This was a pretty big advantage over Chromium, whose developers have consistently refused to support our use cases (which forced us to use the dehydrated certificate witchcraft).  On the negative side, Mozilla has a habit of removing useful API's approximately as fast as we notice new ones.  (Convergence's required API's were removed by Mozilla about a year or two after we started using Convergence, and nczilla's required API's were removed before nczilla even had a proper release, which is why nearly no one has heard of nczilla.)  On the positive side, Mozilla has expressed an apparent willingness to entertain the idea of merging officially supported API's for our use case.  So, I've been hacking around with a fork of Firefox, hoping to come up with something that Mozilla could merge in the future.  Phase 1 of that effort is now complete.

In particular, I've created a C++ XPCOM component (compiled into my fork of Firefox) that hooks the certificate verification functions, and can produce both positive and negative overrides (meaning, respectively, that it can make invalid certs appear valid, and make valid certs appear invalid).  This XPCOM component has access to most of the data that we would want: it has access to the certificate, the hostname, and quite a lot of other data that Firefox stores in objects that the XPCOM component has access to.  Unfortunately, it doesn't yet have access to the full certificate chain (I'm still investigating how to do that), and it also doesn't yet have access to the proxy settings (I do see how to do that, it's just not coded yet).  The full certificate chain would be useful if you want to run your own CA; the proxy settings would be useful for Tor stream isolation with headers-only Namecoin SPV clients.

Performance is likely to be impacted, since this code is not even close to optimized (nor is performance even measured).  I'll be investigating performance later.  Short-term, my next step will be to delegate the override decisions to JavaScript code.  (This looks straightforward, but as we all know, our friend Murphy might strike at any time.)  After that I'll be looking at making that JavaScript code delegate decisions to WebExtensions.  The intention here is to support use cases besides Namecoin's.  The WebExtensions API that this would expose would presumably be useful for DNSSEC/DANE verification, perspective verification, and maybe other interesting experiments that I haven't thought of.

Major thanks to David Keeler from Mozilla for answering some questions in IRC that I had about how the Firefox certificate verification code is structured.

Hopefully I'll have more news on this subject in the next couple weeks.

This work was funded by NLnet Foundation's Internet Hardening Fund.
