---
layout: post
title: "Electrum-NMC: Name Registration GUI"
author: Jeremy Rand
tags: [News]
---

Now that [Electrum-NMC GUI support for updating names]({{site.baseurl}}2018/10/09/electrum-nmc-name-update-gui.html) is a thing, it's time to advance to name registration GUI support.

Whereas the `Renew Name` and `Configure Name...` buttons each map directly to a single sequence of two console commands (`name_update` followed by `broadcast`), which makes their implementation relatively straightforward, *registering* a name is more complicated, due to the two-step procedure in which a salted commitment (`name_new`) is broadcast 12 blocks before the name registration itself (`name_firstupdate`) in order to prevent frontrunning attacks.  Given that the name registration procedure was going to be a bit complicated, it seemed like a good idea to create a new console command for this purpose, so that the GUI can maintain a simple mapping to console commands.

In fact, I ended up creating a few different console commands.  The first console command (`queuetransaction`) is used for storing transactions in the wallet that are intended to be broadcasted in the future once a trigger condition has occurred.  An entry in the transaction queue consists of:

1. Either a transaction ID or a name identifier
2. A depth (in blocks)
3. A raw transaction

When the specified transaction ID (or the most recent transaction for the specified name identifier) attains sufficient confirmations in the blockchain, the raw transaction will be broadcasted.  In order to register a name, the following transaction queue entry would be used:

1. Transaction ID of `name_new`
2. Depth of 12 blocks
3. Raw transaction of `name_firstupdate`

There are some other use cases for the transaction queue, such as automatically renewing names when they're approaching expiration, or automatically registering a name if/when its previous owner lets them expire.  For now, we'll focus on name registration.

Next up was a console command `updatequeuedtransactions`, which examines each of the transaction queue entries, and broadcasts and unqueues each of the entries whose trigger condition has been achieved.  This wasn't too complicated, although I did do some deliberating on when exactly to unqueue a transaction.  In theory, an ElectrumX server could claim to have broadcasted a transaction but not actually do so, and if Electrum-NMC unqueues a transaction in this case, then the transaction will never actually get mined.  A sledgehammer-style workaround here would be to try to re-broadcast each block until Electrum-NMC sees an SPV proof indicating that the transaction has, say, 12 confirmations (indicating that it very likely did get broadcasted and mined).  However, I ended up deciding that this kind of attack is simply out of scope for the transaction queue, since the attack can apply equally well to arbitrary other transactions that get broadcasted.  Solving this attack is probably something better done in upstream Electrum than by whatever hacky and poorly peer-reviewed approach we'd take in Electrum-NMC.  So, we unqueue the transaction as soon as it's broadcast.  Easy enough.

`updatequeuedtransactions` is cool, but we want this to happen every block, automatically.  So the next step was to add a hook that calls `updatequeuedtransactions` whenever a new block arrives.  This should have been simple, but I quickly noticed that whenever this hook resulted in broadcasting a transaction, an assertion error would get logged, and the transaction would never broadcast.  A quick inspection showed that the `broadcast` console command should never be called from the network thread, and the hook was indeed being called from the network thread (where the incoming block event came from).  After a little bit of tinkering, I determined that the simplest approach was just to re-emit the incoming block event to the GUI thread, and then call `updatequeuedtransactions` from the GUI thread's event.

Okay, so the groundwork is laid, now to actually implement a console command for name registration.  In theory, this should be easy: it should consist of `name_new`, `broadcast`, `name_firstupdate`, and `queuetransaction`, right?  Actually, things are a lot more complicated, because if any of the currency inputs to the `name_firstupdate` transaction get spent in the 12-block interval before it gets broadcasted, then the `name_firstupdate` will be rejected as a double-spend.  Hypothetically, I could have fixed this by abusing the address-freezing functionality of Electrum, but there's a better way: pure name transactions.

Pure name transactions are a highly interesting form of Namecoin transaction, where currency is embedded inside a name instead of being kept in a separate input/output.  This works because the 0.01 NMC cost of registering a name is actually enforced as a *minimum* amount of a name output, not an *exact* amount.  You can put, for example, 0.03 NMC into a name output, and you can later withdraw the excess 0.02 NMC by spending that name output.  As long as the amount never drops below the 0.01 NMC minimum, the Namecoin consensus rules don't care.  There are two major use cases for pure name transactions:

1. Reducing transaction size.  Obviously, 1 input and 1 output will yield a smaller transaction than 2 inputs and 2 outputs, which reduces blockchain bloat and transaction fees.  (As far as I know, this use case was first described in a discussion at ICANN58 about Namecoin scalability.)
2. Keeping coins organized.  If you've ever tried to renew more than 25 names in Namecoin Core at once, you might have noticed that you got an error about a long chain of unconfirmed transactions.  This happens because each renewal uses a currency input that's the currency output of the previous renewal, forming a long chain of transactions.  The Namecoin Core error happens because Namecoin Core considers it risky to have a chain of more than 25 unconfirmed transactions (if the first one never got confirmed, all the others would be stuck too).  However, with pure name transactions, each name has its own currency coins, which are temporarily earmarked for use with that name, so operations with different names can't interfere with each other.

The latter use case is what we'll use here.  We create a `name_new` transaction with no currency outputs, but whose name output has an extra 0.005 NMC attached to it.  When we create the `name_firstupdate` transaction, we instruct Electrum-NMC's coin selection algorithm to only use the `name_new` input, and to pay any fees out of the extra 0.005 NMC.  (Coincidentally, I'm pretty sure that Mikhail's Namecoin-Qt client, from the era before Namecoin Core, did the same thing.)  As a result, we can be confident that no accidental double-spends will occur, because we definitely won't be spending the `name_new` output before we broadcast the `name_firstupdate` transaction.  Interestingly, making this work actually needed some minor changes to the Electrum-NMC coin selection algorithm, because parts of the coin selector are not designed to work properly with zero currency inputs being selected.  (Which is understandable, since such a transaction would never be possible in Bitcoin.)

With that, we have a single console command, `name_autoregister`, which does what we want, so now it's time to create a GUI for it.  This was relatively uneventful, but it's notable that I decided to have a separate `Buy Names` tab instead of putting the registration widgets on the existing `Manage Names` tab.  The reasoning for this is that the `Buy Names` tab is a convenient place to show other widgets that don't exist yet, such as giving you the opportunity to atomically trade NMC for a name if the name you want is already registered.

And now, here's your regular fix of screenshots:

![A screenshot of entering a name on the Electrum-NMC Buy Names tab.]({{site.baseurl}}images/screenshots/electrum-nmc/2018-10-17-Buy-Name-Entry.png)

![A screenshot of a name available for registration on the Electrum-NMC Buy Names tab.]({{site.baseurl}}images/screenshots/electrum-nmc/2018-10-17-Buy-Name-Available.png)

![A screenshot of a name already taken on the Electrum-NMC Buy Names tab.]({{site.baseurl}}images/screenshots/electrum-nmc/2018-10-17-Buy-Name-Taken.png)

This work was funded by Cyphrs and NLnet Foundation's Internet Hardening Fund.
