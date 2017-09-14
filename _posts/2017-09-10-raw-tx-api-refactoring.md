---
layout: post
title: Refactoring the Raw Transaction API
author: Jeremy Rand
tags: [News]
---

Several improvements are desirable for how Namecoin Core creates name transactions:

* Register names without having to unlock the wallet twice, 12 blocks apart.
* Coin Control (a method of improving anonymity).
* Specifying a fee based on when you need the transaction to be confirmed.
* Pure name transactions (a method of improving scalability of Namecoin by decreasing transaction size).

All of these improvements, though they seem quite different in nature, have one important thing in common: they need to use the raw transaction API in ways that Namecoin Core doesn't make easy.

## What's the raw transaction API?

Simply put, the raw transaction API is a Bitcoin Core feature that allows the user to create custom transactions at a much lower level than what a typical user would want.  The raw transaction API gives you a lot of power and flexibility, but it's also more cumbersome and, if used incorrectly, you can accidentally lose lots of money.

## Why do those features need the raw transaction API?

The raw transaction API is useful for a few reasons.  First, it lets you create and sign transactions independently from broadcasting them.  That means you can create and sign a `name_firstupdate` transaction before its preceding `name_new` transaction has been broadcast.  Second, it gives you direct control over the inputs, outputs, and fees for the transaction, whereas the standard name commands instead pick defaults for you that you might not like and can't change.

## What's wrong with Namecoin Core's raw transaction API?

It's not raw enough!  First off, the only name transactions it can create are `name_firstupdate`; it can't create `name_new` or `name_firstupdate` transactions.  That rules out handling name registrations, and consequently means that anonymously registering names or controlling the fee for name registrations is a no-go.  Secondly, it can't create `name_update` outputs with a higher monetary value than the default 0.01 NMC.  That rules out pure name transactions.

## How are we fixing this?

We're making an API change.  Instead of trying to stuff name operation data into `createrawtransaction`, where it doesn't belong and where it's extremely difficult to provide the needed flexibility, we're removing name support from `createrawtransaction` and moving it to a new RPC call, `namerawtransaction`.  The new workflow replaces the previous `createrawtransaction` with two steps:

1. Use `createrawtransaction` just like you would with Bitcoin, and specify a currency output of 0.01 NMC for where you want the name output to be.
2. Use `namerawtransaction` to prepend a name operation to the scriptPubKey of the aforementioned currency output.  This has the effect of converting that currency output into a name output.

## What will be the impact?

Users of the raw transaction API will need to update their code.  If you're using the raw transaction API to create currency transactions, then this will actually allow you to delete your Namecoin-specific customizations, since it will be just like Bitcoin again.  If you're using the raw transaction API to create name transactions (this includes people who are doing atomic name trading), you'll need to refactor your `createrawtransaction`-using code so that it also calls `namerawtransaction`.  If you're a hacker who enjoys experimenting, this new workflow will probably be much more to your liking, as it will allow you to do stuff that you couldn't easily do before.  And if you're just an average user of Namecoin-Qt, you'll probably like the new features that this enables, such as easier registration of names, better privacy, and lower fees.

## Who's involved in this work?

* Jeremy Rand wrote a rough spec for the new API
* Daniel Kraft is implementing the changes to the API.
* Jeremy Rand plans to utilize the new API in proof-of-concept scripts for several use cases (including the above 4 use cases).
* Brandon Roberts plans to convert Jeremy's proof-of-concept scripts into GUI features in Namecoin-Qt.

* This work was funded in part by NLnet Foundation's Internet Hardening Fund.

## What might come later?

Maybe Namecoin-Qt support for atomic name trading?  :)
