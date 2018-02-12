---
layout: post
title: "Pruning of Non-scriptPubKey Data in libdohj"
author: Jeremy Rand
tags: [News]
---

Our lightweight SPV client's `leveldbtxcache` mode is the most secure of the various Namecoin lightweight SPV modes.  Its storage requirements aren't too bad either (129.1 MB at the moment for Namecoin mainnet).  However, while 129.1 MB of storage isn't a dealbreaker, it's still a bit borderline on mobile devices.  We can do better.

First, a reminder of how `leveldbtxcache` currently works.  Initially, the IBD (initial blockchain download) proceeds the same way a typical lightweight SPV Bitcoin client (such as Schildbach's Android Bitcoin Wallet) would work: it downloads blockchain headers, aiming for the chain with the most work.  However, at the point when the IBD has reached 1 year ago in the blockchain, it begins downloading full blocks instead of block headers.  The full blocks aren't saved; they're used temporarily for 2 purposes: verifying consistency with the block headers' Merkle root (thus ensuring that no transactions have been censored), and adding any `name_anyupdate` transactions to a LevelDB database that allows quick lookup of names.  After those 2 things have been processed, the full blocks are discarded.  The 129.1 MB storage figure is as low as it is because we're only storing name transactions from the last year (plus block headers, which are negligible in size).

However, there's a lot of data in name transactions that we don't actually need in order to look up names: currency data, signatures, and transaction metadata.

Currency data exists in name transactions because name operations cost a transaction fee, so there will typically be a currency input and a currency output in any name transaction.  We don't need this information in order to look up names.  Signatures are used for verifying new transactions, but are not needed to look up previously accepted transaction data.  Transaction metadata, such as `nVersion` and `nLockTime`, is also not needed to look up names.  There may be other sources of unwanted data too.

To improve the situation, I've just modified `leveldbtxcache` so that, instead of storing full name transactions in LevelDB, it only stores the `scriptPubKey` of the name output.  This includes the name's identifier and value, as well as the Bitcoin-compatible `scriptPubKey` that can be used to verify future signatures.  It's a relatively straightforward change to the code, although it does break backward-compatibility with existing name databases (so you'll need to delete your blockchain and resync after updating).

So, how does this fare?

|                            | **Full Transactions** | **Only `scriptPubKey`** | 
-----------------------------|-----------------------|-------------------------| 
| **Storage Used after IBD** | 129.1 MB              | 63.7 MB                 | 
| **Time Elapsed for IBD**   | ~9 Minutes            | ~6 minutes              | 

Not bad, and we even got a faster IBD as a bonus.  (This suggests that the bottleneck, at least on my laptop running Qubes with an HDD, was storage I/O.)

I've just submitted this change to upstream libdohj.

This work was funded by NLnet Foundation's Internet Hardening Fund.
