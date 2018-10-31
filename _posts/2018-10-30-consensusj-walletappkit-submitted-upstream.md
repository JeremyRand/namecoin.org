---
layout: post
title: "ConsensusJ WalletAppKit Support Submitted Upstream"
author: Jeremy Rand
tags: [News]
---

One of the main reasons that ConsensusJ-Namecoin is still in the [Beta Downloads section]({{site.baseurl}}download/betas/) of Namecoin.org is that it carries several patches against upstream ConsensusJ that haven't yet been upstreamed.  This presents two resultant issues:

1. There are fewer eyes on the ConsensusJ-Namecoin code.
2. ConsensusJ-Namecoin takes some extra time to benefit from new code from upstream, because I need to manually rebase against that new code (and fix whatever conflicts might show up).

Each of these issues reduces the security of ConsensusJ-Namecoin, which means we should fix them by getting our patches upstreamed.  We've been making some progress on this recently, and this week we saw some additional progress, as I've submitted the WalletAppKit support from ConsensusJ-Namecoin upstream.  *Mommy, what's a WalletAppKit?*  The BitcoinJ library, on which ConsensusJ is based, includes quite a few different modes of operation.  The most well-known mode is a mode that downloads blockchain headers and does SPV validation of them.  A lesser-known mode simply opens P2P connections to other Bitcoin nodes and lets the user figure out what to do with those P2P connections.  The latter mode (known as PeerGroup) is utilized as a component of the former mode (known as WalletAppKit).  Currently, upstream ConsensusJ's daemons (`bitcoinj-daemon`) and (`namecoinj-daemon`) use PeerGroup mode, and simply assume (for the purposes of the `bitcoind`-style RPC API) that the peers they've connected to are telling the truth.  ConsensusJ-Namecoin, meanwhile, uses WalletAppKit mode, so that we can be reasonably confident (within SPV's threat model [1]) that the name transactions we're given are actually from the chain with the most work.

Upstream ConsensusJ developer Sean Gilligan has no objection to switching to WalletAppKit, so I've just submitted a pull request to get ConsensusJ-Namecoin's daemons to use WalletAppKit.  If past experience is any guide, it's likely that Sean will want some minor changes made before merging, but I don't expect any major roadblocks to getting this merged.  After WalletAppKit is merged, the most likely next candidate for upstreaming will be the `name_show` RPC call, and associated backend code.  It's likely that the `name_show` upstreaming will focus on `leveldbtxcache` mode, for a few reasons:

1. `leveldbtxcache` mode is by far the most secure SPV mode for Namecoin name lookups (substantially more secure than any SPV clients that exist in the Bitcoin world).
2. `leveldbtxcache` mode has much faster lookups than the other `libdohj-namecoin` name lookup methods, which use an API server.
3. The API server that `libdohj-namecoin` uses by default when not in `leveldbtxcache` mode is WebBTC, which has been down for maintenance for quite a while.  Also, WebBTC is problematic due to its lack of consensus safety (it re-implements the consensus rules instead of relying on Namecoin Core's consensus implementation, and this has caused it to chainfork from the Bitcoin chain before).
4. The API-server security model isn't actually a bad one (it's a lot easier to Sybil the P2P network than to impersonate an API server), but I tend to think that Electrum is a better (and more standardized) approach to the API-server security model than the `libdohj-namecoin` implementation.  It's conceivable that we could combine the two approaches in the future (maybe make `libdohj-namecoin` query an Electrum instance in addition to the P2P network?), but this is not exactly a high priority for our R&D budget given that `leveldbtxcache` mode isn't in any immediate danger of encountering scaling problems.

Hopefully we'll see more progress on this front in the near future.

This work was funded by NLnet Foundation's Internet Hardening Fund.

[1] The SPV threat model makes a security assumption that the chain with the most work is also a valid chain.  This is, of course, not guaranteed to be a correct assumption, although there are some game-theoretic reasons to believe that as long as an economically strong fraction of the network is using full nodes, then SPV is safe for everyone else.  You should definitely run a full node for your Namecoin resolution if you have the ability to do so; you'll be more secure yourself, and you'll also be making the network more secure for anyone who's using an SPV node.
