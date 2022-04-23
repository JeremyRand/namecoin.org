---
layout: post
title: "Electrum-NMC: Name Update GUI"
author: Jeremy Rand
tags: [News]
---

I previously wrote about [creating name transactions in the Electrum-NMC console]({{site.baseurl}}2018/10/06/electrum-nmc-name-transaction-creation.html).  Next up, adding GUI support.

The new `Renew Name` and `Configure Name...` buttons use the previously discussed `name_update` command as their backend, which makes implementation relatively simple, since it's not difficult for GUI functions to access console commands.  I facilitated the `Renew Name` command by making the `value` parameter of `name_update` optional; if not supplied, it will be set to the current value of the name.

However, this introduces an interesting attack scenario: if an ElectrumX server falsely reports the current value of a name you own and you click the Renew button, you would be tricked into signing a `name_update` transaction that contains the malicious value.  Therefore, to mitigate this scenario, `value` is only optional if the latest transaction for the name has at least 12 confirmations.  I can't picture any way that this mitigation would cause real-world UX problems; presumably no one wants to renew a name that was already last updated fewer than 12 blocks ago.

One nice UX improvement in Electrum-NMC compared to Namecoin-Qt is that you can renew multiple names at once: just select more than one name in the `Manage Names` tab, and then click `Renew Name`.  Implementing this was a bit tricky, because Electrum's coin selection algorithm doesn't normally notice that a coin has been spent until it receives a copy from the ElectrumX server, which is usually a few seconds after we broadcast it to the ElectrumX server.  As a result, the coin selection algorithm would try to double-spend the same currency input for each name renewal.  I fixed this by calling `addtransaction(tx)` (which adds a transaction to the wallet) immediately after `broadcast(tx)` returns success, before moving on to the the next name to renew.

It should be noted that renewing multiple names at once will probably reveal to blockchain deanonymizers that the affected names have common ownership.  So, while this feature is potentially useful, it should be used with care.

The `Configure Name...` button and associated dialog were relatively uneventful in terms of bugs.  That said, it did occur to me that it might be better to re-use Namecoin-Qt's `.ui` form files for this purpose instead of re-implementing the dialog in Python.  Unfortunately, Namecoin-Qt's form files are a bit too specific to Bitcoin Core (e.g. they incorporate widgets that only exist in Bitcoin Core, which have different implementations in Electrum).  I think it would be plausible to improve the abstraction of Namecoin-Qt's form files so that we could re-use them in Electrum-NMC and save on duplicated GUI effort; hopefully something will happen there after higher-priority tasks are dealt with in the Namecoin-Qt codebase.

And now, screenshots!

![A screenshot of the "Renew Name" and "Configure Name..." buttons visible in the Electrum-NMC Manage Names tab.]({{site.baseurl}}images/screenshots/electrum-nmc/2018-10-06-Manage-Names-Tab.png)

![A screenshot of the "Configure Name" dialog in Electrum-NMC.]({{site.baseurl}}images/screenshots/electrum-nmc/2018-10-06-Configure-Name-Dialog.png)

This work was funded by Cyphrs and NLnet Foundation's Internet Hardening Fund.
