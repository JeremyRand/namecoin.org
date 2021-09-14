---
layout: post
title: "Namecoin-Qt Buy Names Tab"
author: Jeremy Rand
tags: [News]
---

Now that the Manage Names tab in Namecoin-Qt (which lets you update existing names in your wallet) is [implemented]({{ "/2021/02/16/namecoin-core-name-update-gui.html" | relative_url }}), it's time to move onto the Buy Names tab.  Like the Name Update GUI, this forward-port was pretty uneventful, so rather than boring you with details, here's a screenshot:

![A screenshot of the Buy Names tab in Namecoin-Qt.]({{ "/images/screenshots/namecoin-core/buy-names-2021-08-22.png" | relative_url }})

As you can see, I kept things minimalist to enable faster review.  For example, there is not yet any UI element to tell you if a name already is registered (you'll find out in such cases after clicking the button).  In addition, this UI only triggers the `name_firstupdate` RPC method, not `name_new` (which you'll need to do yourself via the console prior to using the Buy Names tab).  This is because Yanmaani is already working on RPC support for combining `name_new` and `name_firstupdate`; once his work on that is merged, this GUI should be convertible to use his work with a one-liner patch.

The Buy Names tab is expected to be included in Namecoin Core 23.0.

This work was funded by Cyphrs.
