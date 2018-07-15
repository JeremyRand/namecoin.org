---
layout: post
title: "ElectrumX: Name Scripts"
author: Jeremy Rand
tags: [News]
---

ElectrumX is the server component of Electrum.  Unlike the client component, which requires forking to enable altcoins, ElectrumX has altcoin support by default, including Namecoin <a href="#footnote1">[1]</a>.  ElectrumX already supports the AuxPoW features of Namecoin (which is why [only Electrum-NMC needed modifications]({{site.baseurl}}2018/07/09/electrum-nmc-auxpow-verification.html) for that), but name script support required some tweaks to ElectrumX.

The main issue here is that the Electrum protocol requires Electrum to supply scriptPubKey hashes to ElectrumX, for which ElectrumX will then reply with transaction ID's.  This works great when the scriptPubKey can be easily determined by the client (a Bitcoin address can be deterministically converted to a scriptPubKey), but in Namecoin, a scriptPubKey for a name output contains some extra data that Electrum-NMC won't know ahead of time.  Specifically, a `name_update` scriptPubKey includes the `OP_NAME_UPDATE` opcode, the name identifier, the name value, an `OP_2DROP` and `OP_DROP` opcode to empty the opcode, identifier, and value from the stack, and *then* a standard Bitcoin scriptPubKey that corresponds to the address that owns the name.  (A `name_firstupdate` scriptPubKey contains even more stuff.)  An Electrum-NMC client that wants to look up transactions by address won't know the name identifier, and an Electrum-NMC client that wants to look up transactions by name identifier won't know the address of the owner.  As a result, we need to mess with things.

The approach I took was to tweak Namecoin's altcoin definition in ElectrumX to do a few things:

* When hashing a scriptPubKey, first detect whether the scriptPubKey is a name script, and strip away the name script prefix if present.  This leaves only a standard Bitcoin-style scriptPubKey that can be looked up via the standard Electrum protocol.
* Add a secondary method of hashing a scriptPubKey, which detects `name_anyupdate` scripts, and rewrites them into a standard form that's well-suited to name lookups (more on that below).
* When indexing transactions, index the results of both the usual hashing method *and* the secondary method, so that we can reuse ElectrumX's transaction index for both address lookups and name identifier lookups.

How does this standard form work?  Here are the things it changes:

* `name_firstupdate` scripts are converted to `name_update` scripts; the extra data that `name_firstupdate` scripts contain is stripped.
* The name value is replaced with an empty string.  (Technically this means that we use `OP_0` instead of the usual push operation.)
* The Bitcoin-style scriptPubKey after the `OP_DROP` is replaced with `OP_RETURN`.  (Technically this would be interpreted as an unspendable scriptPubKey.)

What are the advantages of this approach?

* Name scripts can be looked up by identifier using the standard Electrum protocol commands; no changes to Electrum's protocol is needed.
* `name_firstupdate` and `name_update` can both be looked up by the same Electrum protocol command.
* The scripts being hashed are unambiguously name scripts, and are not going to appear by accident in other contexts.  It is, of course, certainly possible to deliberately produce a name script whose address is another name script followed by `OP_RETURN`, but Electrum's protocol isn't intended to prevent deliberate collisions anyway.  Any such weird transactions will be detectable when they're downloaded by Electrum-NMC, just like other colliding scripts.  Using `OP_RETURN` as the scriptPubKey also makes it expensive for spammers to produce such collisions, because it means you need to destroy a name (thereby forfeiting your name registration fee) for each collision that you produce.
* The script hash doesn't trivially reveal what name is being looked up; it only reveals a hash.  This improves privacy and security when the name being looked up doesn't actually exist yet in the blockchain (e.g. if you're checking whether a name you want to register already exists).  It should be noted, however, that constructing a rainbow table for this hash function is straightforward, so if it's critical that the names you're looking up not be revealed to the ElectrumX server, you're better off doing lookups via ConsensusJ-Namecoin's `leveldbtxcache` mode instead of the Electrum protocol.  On the other hand, ElectrumX can't actually determine whether the names being looked up are being used for DNS resolution or for registration purposes, so this might disincentivise ElectrumX servers from trying to frontrun registrations, since there'll be a hell of a lot of noise from DNS-resolution-sourced lookups.

Generally speaking, ElectrumX's altcoin abstractions were very pleasant to work with (kudos to Neil Booth and the other ElectrumX contributors on this!), and making these changes wasn't too hard.  Test results:

* ElectrumX takes noticeably longer to sync from Namecoin Core with these changes, but in practical terms the difference isn't a problem: it's 33 minutes instead of 12 minutes on my [Talos II](https://www.raptorcs.com/).  (There might be ways to optimize the speed; I haven't tried to do any optimizations yet.)
* Looking up name transaction ID's by address can be done via the Electrum-NMC console with no changes to Electrum-NMC.
* Looking up name transaction ID's by name identifier can be done via the Electrum-NMC console via the (present in upstream but undocumented) `network.get_history_for_scripthash` command.  However, this requires manually constructing a standard-form scriptPubKey hash from the name identifier, which isn't exactly a fun process for those of us who don't consider Bitcoin script as a native language.
* Looking up unconfirmed name transactions doesn't yet work, because my patch to ElectrumX doesn't yet cover the UTXO index, only the history index.  I haven't tried to fix this yet, and I might not bother for a while, since unconfirmed name transactions aren't really trustworthy unless you're verifying signatures relative to older transactions for that name (such a technique was first proposed years ago by Ryan Castellucci, but to my knowledge no one has actually deployed it yet).
* The procedure for obtaining full transactions from transaction ID's, and for obtaining Merkle proofs for those transactions, is considered out of scope for this work, because Electrum's protocol already supports this, and shouldn't need anything name-specific.  However, it might be interesting to combine those commands with the script hash lookup commands into a single wire protocol command to reduce latency (an interesting area of future work).
* The procedure for parsing name transactions (either for wallet or name lookup purposes) after ElectrumX returns them is considered out of scope for this work, because that should be done by Electrum-NMC; this work only covers changes to ElectrumX.  The needed changes to Electrum-NMC will come later.

I intend to submit these changes to upstream ElectrumX as a PR.

This work was funded by NLnet Foundation's Internet Hardening Fund.

<div id="footnote1">

[1] I actually don't like to refer to Namecoin as an "altcoin", since the term means an "alternative" to Bitcoin, and Namecoin doesn't aim to achieve any of the use cases that Bitcoin does.  However, on a purely technical level, the process of adding Namecoin support to ElectrumX wasn't any different from most altcoins such as Dogecoin, so the umbrella of "altcoin support" does include Namecoin, as much as I dislike the public-relations implications of that label.

</div>
