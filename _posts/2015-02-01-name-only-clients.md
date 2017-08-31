---
layout: post
title: Name-Only Clients
author: Namecoin Developers
tags: [News]
redirect_from:
  - /post/109811339625/
  - /post/109811339625/lightweight-resolvers/
---
Currently, the only options for people wanting to resolve Namecoin names are to deploy a full node or to trust results provided by someone else. It doesn’t have to be this way, though. Like with Bitcoin [lightweight SPV](https://bitcoin.org/en/glossary/simplified-payment-verification) clients which have much smaller local storage requirements can be created for Namecoin. Unlike Bitcoin, we can have “Namecoin Name-Only Clients” that give up support for “Namecoin as a currency” in exchange for better security and even lighter storage requirements.

Without the need to support currency transactions, a Namecoin SPV client can delete block headers that are more than 36,000 blocks old because they are not required to validate “current” (unexpired) name transactions. This data can be stored in a few dozen megabytes. We call this mode “SPV-Current” (SPV-C).

To support SPV-C, a name-only client must be able to query full nodes for transactions by name. This capability can either be added to the Namecoin “wire protocol” or operate as a separate service. Responses include cryptographic proof (using a Merkle hash tree) that the transaction was included in a block which the SPV-C client has headers for.

This basic implementation of SPV-C is vulnerable to a malicious node responding with an older (but non-expired) name transaction in response to a query or censoring the result. The easiest way to combat this is to query multiple full nodes and use the most recent response, but this is fragile. A more robust option would be to use a cryptographic (Merkle) hash tree to represent the list of all “unspent” transactions. This can be used to prove that it is responding with the latest data, since updating a name is done by “spending” the previous update.

The SPV-C client only needs to have the “root” of the hash tree (which is only 32 bytes long) to verify this proof.  By having miners include this root hash in the coinbase transaction of every block, the set of all current names is verified continuously throughout the mining process. Since name operations are marked spent when they expire, the fact of a name transaction is unspent is proof that it is current.  We call this Unspent Transaction Output Tree/Coinbase Commitments (UTXO-CBC).

Once the UTXO-CBC is included in all blocks, name-only clients can then keep track of the current root hash (more accurately, the root hash of the 12th deepest block, as keeping the UTXO set’s current at all blocks in memory is infeasible). When a client queries for a transaction by name, a full node will respond with the name transaction, proof that the transaction was included in a particular block, and that the name data hasn’t been updated by “spending” the transaction. UTXO-CBC constitutes a significant security improvement for SPV-C clients, as it prevents full nodes from returning anything but current name data. This mode is referred to as ‘SPV-C+UTXO-CBC’.

Implementing UTXO-CBC is complicated by a number of problems relating to the formulation of the cryptographic data structure representing the UTXO set.  The most fundamental of these issues is that since new transactions occur continuously, it is important that miners be able to efficiently compute updates to the UTXO set. In other words, it must not be necessary for the entire cryptographic data structure to be recomputed when a new transaction is added to the set. Of course, any solution must preserve the ability to concisely represent the set and the ability to generate small, efficient proofs of inclusion.  Thankfully, a similar UTXO coinbase attestation proposal has been made for Bitcoin, so existing work (such as the Authenticated Prefix Trees proposal) will be transferable to Namecoin.

The final security issue is ensuring that full nodes cannot censor names by simply lying “that name does not exist”. A very similar problem was [solved in DNSSEC](https://en.wikipedia.org/wiki/Domain_Name_System_Security_Extensions#Authenticating_NXDOMAIN_responses_and_NSEC) to provide “authenticated denial of existence” using a sorted list of all names. It’s possible to do this with the same sort of hash tree used to represent all unspent transactions. This commitment is referred to as Unspent Name Output  Non-Existence CBC (UNO NX CBC).  The UNO set is a name-only subset of the UTXO set.

Name-only clients can be improved incrementally, as attacks on even the basic SPV-C design require an attacker to control 100% of the full nodes connected to the target and even then it cannot arbitrarily forge data. Bootstrapping nodes can also attempt to provide a diverse selection of full nodes to clients, increasing the attack cost. However, UTXO CBC and UNO NX CBC reduce the amount of network traffic, reduce latency, and increase security, so they should be implemented eventually.

Hopefully this post has illustrated how both secure and lightweight resolution of Namecoin is not only possible but practical. Once implemented, name-only clients will provide a much better option to those who cannot run a full node versus all other currently available options which either provide no real security and/or require complete trust of a third party.

*This post was slightly edited from its original version upon import to Jekyll, to more accurately reflect consensus among the Namecoin development community.*