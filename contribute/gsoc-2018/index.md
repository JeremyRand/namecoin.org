---
layout: page
title: Google Summer of Code (GSoC) 2018 Project Ideas
---

{::options parse_block_html="true" /}

Namecoin intends to apply for Google Summer of Code 2018 as a mentor organization.

If you are interested in working on one of the following projects, get in touch with us! For low-latency discussion, many of our developers can be reached via [#namecoin-dev]({{site.baseurl}}resources/chat/#namecoin-dev) (we also are present in [#namecoin]({{site.baseurl}}resources/chat/#namecoin)). If you're not able to get an answer on there (which is possible, since several of our developers won't be able to respond during class or work), please post in the [forum](https://forum.namecoin.org/). If you have your own ideas for good Namecoin projects, we would love to hear them and support you with those, too!  All of the project ideas are fairly customizable based on your interests and skills (which is why the "Difficulty" fields aren't completely precise.)  **Please talk to us early in the formation of your proposal; this allows us to give you more useful feedback.**

## Lightweight Client

Namecoin needs a light client, which can be used to register/manage names or get name values trustlessly but without the need to store the full blockchain. There are multiple possible ideas to tackle this, depending on your interests and likes. Some examples of how this might be done: 

* Port an existing lightweight Bitcoin client to Namecoin.  We prefer projects that aim to support the naming features of Namecoin, not just the currency features.  You may find [this list of Bitcoin wallets](https://bitcoin.org/en/choose-your-wallet) to be useful, but be aware that not all of the Bitcoin wallets listed there are suitable.  In particular, clients that are not fully open-source do not qualify; nor do clients that require a server component that is not fully open-source; nor do wallets that require a DRM-enforcing operating system such as Apple iOS; nor do wallets that require fully trusting a third party.  If you are porting a client that uses the BitcoinJ library, you may find [Ross Nicoll's libdohj library](https://github.com/dogecoin/libdohj) to be useful.
* Name database commitment system (for example, a Merkle tree system).  This could make it much more expensive for lookup services to lie about the state of names, make such lies more easily detectable, and allow lightweight clients to only download a subset of the name database.  This has some similarities to the UTXO set commitments that have been proposed in Bitcoin (and used in Electrum's protocol), but Namecoin's needs don't exactly match those of Bitcoin.  You may find [Daniel Kraft's proposal](https://forum.namecoin.org/viewtopic.php?f=5&t=2239) to be interesting.

Adding P2P wire protocol commands may be fine if it helps you in accomplishing this, as long as doing so doesn't cause our codebase to diverge too much from upstream Bitcoin.  Changing blockchain consensus rules is not off the table, but it would require convincing us that doing so has a clear benefit.  You are free to propose a codebase of your choice to work with as well as your own ideas for implementing a light client and discuss your choice with the community.  You may find the following writeups by Hugo Landau interesting, although take them with a grain of salt as they are somewhat outdated: [Namecoin Deployment Types](https://github.com/hlandau/ncdocs/blob/master/nctypes-intro.md) and [State of Namecoin](https://github.com/hlandau/ncdocs/blob/master/stateofnamecoin.md).

**Difficulty**: Medium-Hard

**Requirements**: If porting an existing Bitcoin client, a strong understanding of how Bitcoin and Namecoin work.  If working on a name database commitment system, familiarity with commitment systems such as those using Merkle trees.

**Expected Outcomes/Deliverables**: A light client that is usable and supports name transactions.

**Possible Mentors**: Jeremy (confirmed), Brandon (confirmed) <!-- Daniel? -->

## Domain Name Integration with Additional Platforms/Applications

Namecoin currently supports integration with the DNS resolvers on desktop OS's such as GNU/Linux, Windows, and macOS.  Integration with the Tor anonymity network on desktop OS's (with domain names pointing to IP addresses, DNS names, and onion services) is also working.  However, there are several other applications and environments where Namecoin domain names would be beneficial.  Some of these are:

* The Android mobile OS.  This might also include porting an Android Light Client to Namecoin.
* The [I2P](https://geti2p.net/) anonymity network (names point to IP addresses, DNS names, and eepsites).  (Note: the [Kovri](https://getkovri.org/) developers are interested in this.)
* The [Freenet](https://freenetproject.org/) anonymous hosting network (names point to freesites).
* The [ZeroNet](https://zeronet.io/) decentralized hosting network (names point to ZeroNet sites).  ZeroNet already has partial support for Namecoin but currently uses a centralized server to look up names.
* The [IPFS](https://ipfs.io/) decentralized hosting network (names point to IPFS sites).

You are free to integrate Namecoin domain names into any additional platform(s) or application(s) of your choice, as long as they do not enforce DRM (this means Apple iOS is not acceptable).  In many cases, it is likely that you would benefit from talking to the developers of whatever application you would be working with (e.g. if you're implementing Namecoin support in I2P, you might want to talk to the developers of an I2P client implementation).

**Difficulty**: Easy-Medium

**Requirements**: Familiarity with whatever platform(s) or application(s) you would be integrating Namecoin with.

**Expected Outcomes/Deliverables**: A working integration of Namecoin domain names with the platform(s) or application(s) of your choice.

**Possible Mentors**: Jeremy (confirmed), anonimal (if integrating Namecoin with Kovri; confirmed)

## TLS Integration with Additional Platforms/Applications

Namecoin currently supports integration with TLS implementations (as a decentralized alternative to certificate authorities) on a small number of platforms and applications: positive overrides (self-signed certs are allowed if they match the Namecoin blockchain) for Windows and (on GNU/Linux) NSS, and negative overrides (CA-issued certificates are disallowed if they don't match the Namecoin blockchain) for Chromium.  However, our existing support is rather hacky and has some disadvantages, and we'd love to support more platforms.  Some examples of things we'd like to see support Namecoin for TLS:

* Operating Systems
    + Windows
    + macOS
    + GNU/Linux
    + Android
    + Any other OS
* Any major web browser (except Firefox -- we already have a developer working on that)
* Libraries
    + OpenSSL
    + NSS
    + GnuTLS
    + Any other TLS library
* TLS Shims
    + [Cert-Shim](https://bitbucket.org/uf_sensei/cert-shim)
    + [TLS Pool](https://github.com/arpa2/tlspool)
* Languages' Standard Libraries
    + Go
    + Python
    + Java
    + Any other language's standard library
* Servers
    + CertBot
    + Caddy

It might also be interesting to simulate HSTS by automatically redirecting HTTP to HTTPS for Namecoin domain names that have a TLS cert listed in the blockchain.  This would prevent sslstrip-style attacks even if an HSTS header hasn't been received yet.

**Difficulty**: Medium-Hard

**Requirements**: Familiarity with TLS certificates and how they're handled by whatever platform(s)/application(s) you choose to work with.

**Expected Outcomes/Deliverables**: A working integration of Namecoin for certificate verification with the platform(s) or application(s) of your choice.

**Possible Mentors**: Jeremy (confirmed)

## Identity Integration with Additional Applications

Namecoin could be used for identities with a variety of protocols and applications.  Some of these are:

* Website single sign-on.  Daniel Kraft's NameID is an interesting proof-of-concept of this, but it is questionable whether OpenID or a Firefox extension are the best ways to do this.  For example, TLS CCA (client certificate authentication) and FIDO have been suggested as more standard alternatives.
* OpenSSH client login.
* [Bitmessage](https://bitmessage.org/).  Daniel Kraft implemented a proof-of-concept that is now part of Bitmessage, but it could be significantly improved, e.g. by implementing reverse lookups (Bitmessage addresses to Namecoin identities).  Be careful about blockchain bloat; it might be more scalable to implement reverse lookups in the Bitmessage protocol rather than the Namecoin blockchain.
* OTR (Off The Record Messaging) and OMEMO.  Daniel Kraft implemented a proof-of-concept for Pidgin, but his code is obsolete, and Pidgin itself is no longer considered particularly safe.  There are lots of IM clients that support OTR or OMEMO; adding Namecoin support to one of them would be beneficial.
* OpenPGP.  Phelix implemented a proof-of-concept of a Namecoin OpenPGP keyserver, but it is based on outdated code and might benefit from a fresh attempt.
<!-- * Ricochet.  **TODO: Check with Special about whether this is a good idea for 2018.**-->
<!-- * Tox.  **TODO: Check with Tox devs about whether this is a good idea for 2018.**-->
<!-- * Ring.  **TODO: Check with Ring devs about whether this is a good idea for 2018.**-->
<!-- * Matrix.  **TODO: Check with Matrix devs about whether this is a good idea for 2018** -->
* Cryptocurrency addresses.  May be relevant to the work by Netki and by [Monero's OpenAlias project](https://www.openalias.org/).

**Difficulty**: Easy-Medium

**Requirements**: Familiarity with whatever protocol(s) or application(s) you would be integrating Namecoin with.

**Expected Outcomes/Deliverables**: A working integration of Namecoin identities with the protocol(s) or application(s) of your choice.

**Possible Mentors**: <!-- Daniel? --> Jeremy (confirmed)

## Name Trading UI

Namecoin's protocol supports atomic name trades, in which Alice can purchase a name from Bob without counterparty risk.  This could help reduce issues with squatted names.  You could make this functionality available in the UI in a simple way, and possibly investigate a method for buyers and sellers to find each other (e.g. a decentralized bulletin board).  Both interactive trades (where both parties select and confirm each other) and non-interactive trades (where a trade offer can be posted publicly and another user can accept the offer without further interaction) are possible; you can implement either (or both).  You may find [Phelix's ANTPY](https://github.com/phelixnmc/antpy) and [Ryan Castellucci's nametrade](https://github.com/ryancdotorg/nametrade) implementations to be interesting.

You are free to implement a name trading UI in any Namecoin wallet that you choose.  (Since Namecoin Core is the only currently available wallet application, this would mean either implementing your name trading UI into Namecoin Core or combining some name trading UI tasks with creating a Lightweight Client.)

**Difficulty**: Medium

**Requirements**: Familiarity with UI design in whatever language and UI framework your chosen wallet application uses.  (For example, Namecoin Core uses C++ with Qt.)

**Expected Outcomes/Deliverables**: Modified Namecoin wallet software with your atomic name trading UI.

**Possible Mentors**: Brandon (confirmed), Jeremy (confirmed) <!-- Daniel? -->

## Anonymity

Namecoin suffers the same problems with traceability (lack of anonymity) of coin flows that Bitcoin does, plus additional linkability given the data attached to name transactions.  When Namecoin is used for its stated purpose to provide censorship-resistant, MITM-resistant browsing and registration of domains for possibly controversial purposes (e.g. under repressive regimes), this may be even a bigger problem than with Bitcoin.  You could improve this situation in several ways:

* Improve usability of registering a name anonymously by automating a purchase of namecoins using an anonymous currency like Monero or Zcash as part of the name registration process.  This reduces the risk of private data leaking while purchasing namecoins (e.g. banking details at an exchange, or Bitcoin blockchain analysis data).  Integrating with a decentralized exchange such as [Bisq](https://bisq.network/) might be a useful way to do this (this might also make it easier to register Namecoin names with national currencies).  Future advances in blockchain technology such as atomic cross-chain trades might be applicable, so try to make sure that your code could be reused with as few changes as possible for such use cases once they're ready.
* Keep each registered name separate on the blockchain.  This reduces the risk of private data leaking in the Namecoin blockchain.  This might be done using the Coin Control or HD Wallet features of wallet software.
* Improve the ability for Namecoin-related software to run safely when routed over Tor or other anonymity networks.  This might include testing for and fixing proxy leaks, application-layer protocol leaks, timing metadata leaks, and incorrect or missing stream isolation, among other things.  It's okay if this work includes some fixes to upstream software (e.g. Bitcoin wallet software), but a significant component of your project should be Namecoin-specific.

You are free to choose any anonymity improvements you wish in any Namecoin-related software of your choice.  <!-- **TODO: are the Bisq people and/or the Monero people interested in mentoring something?  (Asked in #bisq 2018 Jan 18.)** -->

**Difficulty**: Medium-Hard

**Requirements**: Familiarity with Bitcoin's anonymity properties/attacks.

**Expected Outcomes/Deliverables**: Modified Namecoin software with your anonymity improvements.

**Possible Mentors**: Jeremy (confirmed) <!-- Daniel? -->

## Scalability: Segregated Name Values

The data attached to names in Namecoin is permanently part of the blockchain, which is bad for scalability.  Jeremy Rand wrote a rough proposal called [Segregated Name Values (SegVal)](https://forum.namecoin.org/viewtopic.php?f=5&t=2482), which would allow name data to be discarded from the network after it has expired, thereby significantly improving scalability.  You could implement SegVal and deploy it on a Namecoin testnet, for merging later to the production Namecoin mainnet.

**Difficulty**: Hard

**Requirements**: Familiarity with C++ and the Bitcoin Core codebase.

**Expected Outcomes/Deliverables**: SegVal deployed on a Namecoin testnet.

**Possible Mentors**: <!-- Daniel? --> Jeremy (confirmed)

## Block Explorer

Namecoin would benefit from a high-quality libre block explorer with support for name transactions.  Brandon Roberts has already ported [Bitcore](https://bitcore.io/) to Namecoin, which might make it feasible to port the [Insight block explorer](https://github.com/bitpay/insight-api) to Namecoin.

**Difficulty**: Easy

**Requirements**: Basic familiarity with a Bitcoin block explorer implementation.

**Expected Outcomes/Deliverables**: A Namecoin block explorer with support for name transactions.

**Possible Mentors**: Brandon (confirmed), Jeremy (confirmed)

## Packaging Improvements

Namecoin's various subprojects would benefit from improvements in the packaging process.  For example:

* GNU/Linux packages for Namecoin Core, ncdns, lightweight clients, etc.
* [Reproducible builds](https://reproducible-builds.org/) for ncdns, lightweight clients, etc.  (Namecoin Core binaries are already reproducible, but we'd love to see reproducible GNU/Linux packages for Namecoin Core.)

**Difficulty**: Easy-Hard

**Requirements**: Familiarity with build systems, packaging, and/or reproducible builds.

**Expected Outcomes/Deliverables**: Modified Namecoin software build scripts with your improvements; possibly also deployed PPA/COPR-style packages or official distribution packages if you choose to do so.

**Possible Mentors**: Jeremy (confirmed)

## Hardening/Sandboxing Improvements

Various hardening and sandboxing technologies exist, e.g. [AppArmor](https://en.wikipedia.org/wiki/AppArmor), [Bubblewrap](https://github.com/projectatomic/bubblewrap), [Qubes](https://www.qubes-os.org/)'s AppVM's, and [Subgraph](https://subgraph.com/sgos/)'s Oz.  It would be interesting to utilize some of these with software from the Namecoin ecosystem.

**Difficulty**: Easy-Medium

**Requirements**: Familiarity with the hardening/sandboxing technology or technologies that you plan to use.

**Expected Outcomes/Deliverables**: Modifications to the Namecoin-related software of your choice to use the hardening/sandboxing technology or technologies of your choice.

**Possible Mentors**: Jeremy (confirmed)

## Your Own Namecoin Use or Enhancement

Namecoin can be used for lots of novel and interesting use cases, and existing use cases are open for improvement.  Propose your own use case or improvement, and discuss it with us!  Anecdotally, we've heard from other GSoC mentor organizations that the most successful projects were ones that a student came up with themselves rather than took from a suggestion list, so don't be afraid to be creative.  Also note that there's nothing wrong with working on more than one of the categories we've suggested.  For example, anonymous users often have bandwidth constraints, so combining Anonymity with a Lightweight Client might make sense; just be careful not to choose a bigger scope than you can expect to do over the summer.
