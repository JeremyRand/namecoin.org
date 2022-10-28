---
layout: post
title: "Changes to name_show in Electrum-NMC Protocol v1.4.3"
author: Jeremy Rand
tags: [News]
---

In my 36C3 presentation (seems so long ago...), I mentioned some future changes I wanted to make to the Electrum-NMC protocol.  Some of these changes have recently been implemented:

## Verifying Unconfirmed Transactions

For security reasons, Electrum-NMC has historically required 12 confirmations before it accepts a name transaction.  Unfortunately, waiting 2 hours (on average) for a name update to take effect is not great UX, and also introduces its own security issues (e.g. it means that TLS revocations are unnecessarily delayed).  When evaluating security of name transactions, there are two different attacks that we need to consider:

* Hijacking attacks
* Double-spend attacks

Hijacking attacks occur when a malicious miner mines a Namecoin block with an invalid scriptSig, which (if accepted) would allow them to hijack an arbitrary name without any participation from the name owner.  In contrast, double-spend attacks occur when a malicious name owner signs two different name updates with the same input, which (if accepted) would allow one of those name updates to be accepted even though only the other update is part of the longest blockchain.

In practice, while any name could be targeted by a hijacking attack, only names that have recently been transferred to a new owner can be targeted by a double-spend attack.  (Double-spending a name that you still own doesn't really confer any benefit.)  Since the vast majority of names have not been transferred in the last 12 blocks, if we can prevent hijacking attacks, we can accept the risk of double-spend attacks, since the latter can easily be mitigated by waiting 12 blocks from receiving a name transfer before using that name for critical purposes.

Can we do this?  Ryan Castellucci proposed many years ago that we could verify the scriptSigs of recent name updates, chaining them back to a previous name update that has sufficient DMMS confirmation (i.e. 12 blocks).  I have now implemented this in Electrum-NMC Protocol v1.4.3; Electrum-NMC will retrieve recent updates to a name (from 1 to 11 confirmations) and verify scriptSigs to chain them back to the most recent update with at least 12 confirmations.  The result is that name updates now take effect in 1 block (~10 minutes) instead of 12 blocks (~2 hours).

## Checkpoint Verification

One of the inherent problems with the SPV threat model is that it only verifies DMMS signatures (attached to blocks), not scriptSigs (attached to transactions).  This means that an attacker with a majority of hashrate can mine invalid blocks (e.g. that hijack names) that will appear valid to SPV clients.  Full nodes are, of course, immune to this, since they check scriptSigs on all transactions.

Unfortunately, full nodes have a much longer IBD duration, and also are much more resource-intensive.  Can an SPV client verify scriptSigs on the transactions it cares about?  In Bitcoinland, this is not really feasible, because the fungible nature of bitcoins means that you'd have to verify every transitive input to every transaction you care about, which would probably be exponential with respect to the verification depth.  However, in Namecoinland, we don't have this problem.  The nonfungible nature of Namecoin names means that the number of transitive inputs grows linearly, not exponentially.

Thus, we can reuse the scriptSig chaining trick to chain the latest confirmed name update back to a previous update that was committed to in the checkpoint (if such an update exists).  This means that in order to hijack a name that existed at the checkpoint height, an attacker would need to compromise both a majority of hashrate **and** the full node used to generate the checkpoint.  I've implemented this in Electrum-NMC Protocol v1.4.3 as well; Electrum-NMC will retrieve all updates for a name going back to just before the checkpoint height, and verify the chain of scriptSigs.

## One Round Trip

Electrum-NMC's protocol has always been inefficient for `name_show`, because it needed to issue 4 commands to the ElectrumX server per name:

* Get the latest name update txid.
* Get the transaction preimage.
* Get the Merkle proof tying the txid to the block header.
* Get the Merkle proof tying the block header to the checkpoint.

The latter 3 of those could be issued in parallel, but that still means a best case of 2 round trips per lookup (and the best case was not always achieved in practice).  Especially over Tor, this extra latency is a major UX problem.  This inefficiency was a result of the upstream Electrum protocol for Bitcoin not being designed for Namecoin's use cases.  As of Electrum-NMC Protocol v1.4.3, there is a dedicated command for `name_show`, which combines the above 4 commands into 1, thus cutting down the lookup time to a single round trip.  The above scriptSig-related features are also integrated into the dedicated command, so retrieving the extra transactions does not incur any extra round trips.

## Future Optimizations

Electrum-NMC Protocol v1.4.3 also allows the ElectrumX server to send hint data for names other than the requested one.  For example, if `d/wikileaks` imports from `dd/wikileaks`, the server might return data for both `d/wikileaks` and `dd/wikileaks` when the client requests the former, thus saving a round trip if the client would have asked for the latter name next.  At the moment, the hint data is left unpopulated, but we should be able to start populating it in the future without needing another protocol change.

## Deployment

Electrum-NMC and upstream ElectrumX have both merged Protocol v1.4.3, so we're now waiting for server operators to update.  Once we've determined that enough server operators have updated, we expect to flip a switch in the Electrum-NMC source code that enables the new protocol features.  This functionality should also work out of the box in other ElectrumX-supported coins that use a name index, which in practice means Xaya.

This work was funded by NLnet Foundation's NGI0 Discovery Fund.
