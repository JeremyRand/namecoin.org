---
layout: post
title: "Reproducible Builds of NSS certutil via Cross-Compiling with rbm"
author: Jeremy Rand
tags: [News]
---

In a previous post [where I introduced `tlsrestrict_nss_tool`]({{site.baseurl}}2018/03/26/integrating-cross-signing-name-constraints-nss.html), I mentioned that NSS's `certutil` doesn't have official binaries for Windows, and that "At some point, we'll probably need to start cross-compiling NSS ourselves, although I admit I'm not sure I'm going to enjoy that."  Well, we've reached that point, and it was an interesting adventure.

Initially, I looked at the NSS build docs themselves, and was rather annoyed to find that there's no documentation about how to cross-compile NSS.  To make matters worse, the only results I could find by Startpaging were people saying that they couldn't figure out how to cross-compile NSS (including some well-known software projects' developers).

However, it just so happens that there's a very high-profile free software project whom I was certain is definitely cross-compiling NSS: The Tor Project.  Tor cross-compiles Firefox (including NSS) as part of their Tor Browser build scripts, so it seemed near-certain that their build scripts could be modified to build `certutil`.  However, Tor's build scripts are rather nonstandard, since they build everything with rbm.  rbm is a container-based build system that's superficially similar to the Gitian build system that Bitcoin Core and Namecoin Core use (indeed, The Tor Project used to use Gitian before they migrated to rbm).  I've been intending to get my feet wet with rbm for quite a while now, so this seemed like a great excuse to play with rbm a bit.

First off, I wanted to build Firefox in rbm without any changes.  This was actually quite easy -- The Tor Project's documentation is quite good, and I didn't run into any snags (besides the issue that I initially assigned too little storage to the VM where I was doing this -- The Tor Project should probably document the expected storage requirements).  The build command I used was:

~~~
./rbm/rbm build firefox --target nightly --target torbrowser-windows-i686
~~~

Next, I looked at Tor's Firefox build script... and I was delighted to see that Tor is *already* building `certutil`.  In fact, you can [download `certutil` binaries from The Tor Project's download server](http://rqef5a5mebgq46y5.onion/torbrowser/) right now!  (You want the `mar-tools-*.zip` packages.)  Except... their build script discards the `certutil` binaries for all non-GNU/Linux targets.  How sad.

Modifying the build script to also output `certutil.exe` for Windows was reasonably straightforward -- rbm even worked without erroring on the first try.  I did, however, need to try for a few iterations to make sure that I was outputting all of the needed `.dll` files.  However, once I had all the required `.dll` files, a rather odd symptom occurred when I tested it on a Windows machine.  When I ran `certutil.exe` from a command prompt, it would immediately exit without printing anything.  Stranger, when I double-clicked `certutil.exe` in Windows Explorer, it didn't even pop up with a command prompt window before it exited.  In addition, I noticed that if I passed command line arguments to `certutil.exe` telling it to create a new database, it actually did create the database -- but it still didn't display any output.

This seemed to indicate that something was wrong not with `certutil.exe`'s actual functionality, but with its PE metadata: Windows was probably treating it as a GUI application rather than a console application.  Checking the PE metadata confirmed this: Tor's build scripts were producing a `certutil.exe` whose PE metadata was marking it as a GUI application.  Some more quick searching revealed [a StackOverflow post](https://stackoverflow.com/questions/2435816/how-do-i-poke-the-flag-in-a-win32-pe-that-controls-console-window-display/14806704#14806704) providing a short Python2 script that could edit that part of the PE metadata.  I ran that script against `certutil.exe`... and now `certutil.exe` works properly.  Yay!

The lack of `certutil.exe` binaries was one of the major blockers for releasing negative TLS certificate overrides for Firefox on Windows.  Now that this barrier is behind us, I can get around to testing `tlsrestrict_nss_tool` on Windows, and hopefully do a release (with NSIS installer support).  And as a side bonus, `certutil.exe` builds reproducibly, and I've now gotten some experience with rbm (meaning that reproducible builds for ncdns and our other Go software may be coming soon).

This work was funded by NLnet Foundation's Internet Hardening Fund.
