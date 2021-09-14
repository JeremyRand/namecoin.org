---
layout: post
title: "Upstream btcd Has Merged Last Outstanding Namecoin Patch"
author: Jeremy Rand
tags: [News]
---

Good news: upstream btcd has merged the last outstanding patch that Namecoin was applying for our usage of btcd's JSON-RPC client in ncdns.  This means that Namecoin's fork of btcd will be discontinued, and as of the next btcd release (v0.22.1), ncdns will switch to using an unpatched upstream btcd.  Using unpatched upstream btcd will decrease our maintenance effort and improve code quality.

Huge thanks to Torkel Rogstad, Anirudha Bose, and John Vernaleo from upstream btcd for code review.
