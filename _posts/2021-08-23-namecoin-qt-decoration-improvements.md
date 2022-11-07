---
layout: post
title: "Namecoin-Qt Decoration Improvements"
author: Jeremy Rand
tags: [News]
---

I've been improving the UX of decorations applied to transactions in Namecoin-Qt.  For comparison, here's what it looked like before the improvements:

![A screenshot of the Transactions tab in Namecoin-Qt.]({{ "/images/screenshots/namecoin-core/transactions-2021-08-02.png" | relative_url }})

And after:

![Another screenshot of the Transactions tab in Namecoin-Qt.]({{ "/images/screenshots/namecoin-core/transactions-2021-08-09.png" | relative_url }})

As you can see, jargon such as `NAME_FIRSTUPDATE` has been replaced with more user-friendly terminology such as `Name registration` and has been moved to the `Type` column (replacing the redundant `Name operation` text).  Name updates that do not change the value are now marked as `Name renewal`, which more precisely conveys the nature of such transactions (this required some plumbing, as previously the transaction decoration code did not have access to the previous value of a name transaction).  Additionally (not pictured above), name transfers are now decorated appropriately.

These improvements are expected to be included in Namecoin Core 23.0 (22.0 has already branched from master and should be released soon).
