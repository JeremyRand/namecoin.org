---
layout: post
title: "Namecoin-Qt: Export, Name Count, and Buy Names Feedback"
author: Jeremy Rand
tags: [News]
---

I implemented a few more Namecoin-Qt features.  The Manage Names tab now includes both an Export button and a name counter:

![A screenshot of the Manage Names tab in Namecoin-Qt.]({{ "/images/screenshots/namecoin-core/manage-names-2021-09-14.png" | relative_url }})

The Buy Names tab also now gives real-time feedback on which names are available:

![A screenshot of the Buy Names tab in Namecoin-Qt, when the name is available.]({{ "/images/screenshots/namecoin-core/buy-names-available-2021-09-14.png" | relative_url }})

![A screenshot of the Buy Names tab in Namecoin-Qt, when the name is taken.]({{ "/images/screenshots/namecoin-core/buy-names-taken-2021-09-14.png" | relative_url }})

This is a good example of a UX benefit that only full-node users get: because name lookups in Namecoin Core don't generate network traffic, they are fast enough that they can be performed every time a character is typed in the name field, without making the user click anything.

I'm not leaving RPC users behind either: the `name_show` RPC method now yields more specific error messages when a name doesn't exist, depending on whether it is expired or never existed.  This should make things more convenient for human RPC users.  The error codes haven't changed, so this shouldn't break any software.

These improvements are expected to be included in Namecoin Core 23.0.

This work was funded by Cyphrs.
