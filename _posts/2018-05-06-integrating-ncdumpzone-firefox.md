---
layout: post
title: "Integrating ncdumpzone's Firefox TLS Mode into ncdns"
author: Jeremy Rand
tags: [News]
---

I [discussed in a previous post]({{site.baseurl}}2018/02/20/ncdumpzone-firefox.html) some experimental work on making ncdumpzone output a Firefox certificate override list.  At that time, the procedure wasn't exactly user-friendly: you'd need to run ncdumpzone from a terminal, redirect the output to a file, close Firefox, delete whatever existing `.bit` entries existed in the existing Firefox certificate override file, append the ncdumpzone output to that file, and relaunch Firefox.  I've now integrated some code into ncdns that can automate this procedure.

One of the trickier components of this was detecting whether Firefox was open.  Firefox's documentation claims that it uses a lockfile, but as far as I can tell Firefox doesn't actually delete its lockfile when it exits (and I've seen similar reports from other people).  Eventually, I decided to just watch the contents of my Firefox profile directory (sorted by Last Modified date) as Firefox opened and closed, and I noticed that Firefox's sqlite databases produce some temporary files (specifically, files with the `.sqlite-wal` and `.sqlite-shm` extension) that are only present when Firefox is open.  So that's a decent hack to detect that Firefox is open.

Given that, ncdns now creates 2 extra threads: `watchZone` and `watchProfile`.  `watchZone` dumps the `.bit` zone with ncdumpzone every 10 minutes, and makes that data available to `watchProfile`.  (Right now, ncdumpzone is called as a separate process, which isn't exactly ideal -- a future revision will probably refactor ncdumpzone into a library so that we can avoid this inefficiency.)  `watchProfile` waits for Firefox to exit (it checks at 1 Hz), and then loads Firefox's `cert_override.txt` into memory, removes any existing `.bit` lines, appends the data from `watchZone`, and writes the result back to Firefox's `cert_override.txt`.

These 2 new threads in ncdns are deliberately designed to kill ncdns if they encounter any unexpected errors.  This is because, if we stop syncing the `.bit` zone to the Firefox override list, Firefox will continue trusting `.bit` certs that might be revoked in Namecoin.  Therefore, it is important that, in such a situation, `.bit` domains must stop resolving until the issue is corrected.  Forcing ncdns to exit seems to be the least complex way to reliably achieve this.

These changes significantly improve the UX of positive TLS certificate overrides for Firefox.  A pull request to ncdns should be coming soon.

This work was funded by NLnet Foundation's Internet Hardening Fund.
