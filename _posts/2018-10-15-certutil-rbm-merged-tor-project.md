---
layout: post
title: "NSS certutil Windows/macOS rbm Build Scripts Merged by The Tor Project"
author: Jeremy Rand
tags: [News]
---

As I mentioned earlier, [I submitted some patches]({{site.baseurl}}2018/09/27/certutil-windows-rbm-submitted-tor-project.html) to The Tor Project for building NSS certutil binaries for Windows and macOS as part of Tor Browser's rbm build scripts.  I'm happy to report that after a (quite well-justified) delay, and after some (quite reasonable) mild edits were requested and made, the Tor developers have merged my patches.

This has benefits to multiple parties.  It benefits Namecoin since it means we get trustworthy certutil binaries for our Namecoin TLS releases [1].  It benefits the broader NSS ecosystem on Windows and macOS, since (among other things) it means that Firefox users on Windows and macOS won't need to download random binaries linked from StackExchange or forums, or self-compile NSS, just in order to add certificates to their trust store from the command line.  (Based on the number of such forum threads I found via a cursory web search, there are a lot of such users.)  It benefits Tor, since it means that users who wouldn't otherwise be interacting with the Tor ecosystem will now have a reason to do so, which gives Tor some free publicity.  And it benefits the reproducible build ecosystem, because many users who previously assumed they'd either have to risk downloading malware or build from source will now be learning about the benefits of reproducible builds for the first time (and will hopefully start demanding similar security guarantees from the developers of other software they use).

This is why, at Namecoin, we try to get our work upstreamed as much as possible -- we aim for maximum public benefit across the ecosystem, rather than maintaining forks of software for our own limited use cases.  Kudos to the Tor devs for the excellent code review experience.  I'm looking forward to getting more patches upstreamed to Tor in the future.

This work was funded by NLnet Foundation's Internet Hardening Fund.

[1] Of course, this is partially negated by the fact that some ongoing R&D happening at Namecoin suggests that we may be able to ditch the certutil dependency, in favor of some other approaches with less attack surface.  But I'll save that for another post.
