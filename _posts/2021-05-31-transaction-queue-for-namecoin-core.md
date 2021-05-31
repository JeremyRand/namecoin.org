---
layout: post
title: "Transaction queue for Namecoin Core"
author: yanmaani
tags: [News]
---

Lately, work aiming to simplify the RPC API for name management has been proceeding. This is done both for the sake of improvement itself, and to make it easier to write GUIs for Namecoin.

One unique element in Namecoin is the extensive use of time-dependent transactions. This is comparatively rare in Bitcoin. Either you want to send money or you don't want to send money, but the "when" is usually ASAP. However, in Namecoin, it's pretty common to, for instance, want to renew a name *when* it's close to expiring, or register a name *when* its `NAME_NEW` input has matured.

Historically, this has been a bit of an annoyance. Transactions can only* be broadcast if they have a good chance of making it into the next block, and not all transactions are intended to be broadcast immediately. The way users had to deal with it, then, was to manually keep track of what transactions they wanted broadcast and when.

This isn't suitable. It's bad enough for direct users, but it also makes writing auxilliary software a pain, since the obligation will fall upon it to manage a lot of state. For example, the old name management GUI had to have a series of undocumented internal hooks into the wallet database.

For this reason, a transaction queue has now been added to Namecoin to explicitly manage all that state. It was written by Brandon Roberts, Jeremy Rand, and yanmaani, who is the author of this post.  The implementation is actually shockingly simple:

1. Keep a list of transactions we'd like to broadcast
2. Each block, try to broadcast everything
3. Remove everything that made it into the mmepool

The initial plan was to have a very baroque API, where users could enter transactions and have them broadcast either when a name reached a certain age or when a transaction had a certain height. However, *after already having implemented all this*, it was realized that Bitcoin's existing locking facilities could already cover our needs, and so I took it out. Wasting the effort was a bummer, but you can't be sentimental like that. In the end it made for a better API and cleaner code. Such is life.

The API for the transaction queue is thus very simple:

`queuerawtransaction <hex>` queues a transaction to be broadcast as soon as it can, and returns the txid on success. There is a basic sanity check to try and ensure the transaction at least theoretically could be broadcast at some point in the future, but there's no guarantee. Note that it's on you to properly lock the transaction, or else it will be broadcast immediately. If you want to update a name when it's 35000 blocks old, make sure to set nSequence on the name input to 35001.

`dequeuerawtransaction <txid>` removes a transaction from the queue.

`listqueuetransactions` lists all the queued transactions. Note that this may also include transactions queued by RPC calls.

These changes will be available in Namecoin Core starting version 0.22.

This change was a prerequisite for `name_autoregister`, an RPC call to register a name in one step without having to manually call `name_firstupdate`, regarding which we endeavor to have an update in the next few days. That RPC call is the final pre-requisite for completing the new name management GUI.

This work was funded by NLnet Foundation's NGI0 Discovery Fund, which is funded by the European Commission, the executive branch of the European Union.

* Actually, this is only true in Bitcoin. In Namecoin, presumably to take the edge off this wart, there's a special (somewhat ungainly) exemption for `name_firstupdate`, which was the cause of very serious bugs in the past, such as the [June 2018 incident]({{ "/2018/07/08/brownout-june-17-20.html" | relative_url }}).
