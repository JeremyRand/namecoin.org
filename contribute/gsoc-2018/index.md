---
layout: page
title: Google Summer of Code (GSoC) 2018 Project Ideas
---

{::options parse_block_html="true" /}

Namecoin intends to apply for Google Summer of Code 2018 as a mentor organization.

**This Ideas Page is still under construction, and contains some relics from 2015, 2016, and 2017. Check back soon for updates.**

If you are interested in working on one of the following projects, get in touch with us! For low-latency discussion, many of our developers can be reached via #namecoin-dev on Freenode IRC (we also are present in #namecoin). If you're not able to get an answer on there (which is possible, since several of our developers won't be able to respond during class or work), please post in the [forum](https://forum.namecoin.org/). If you have your own ideas for good Namecoin projects, we would love to hear them and support you with those, too!  All of the project ideas are fairly customizable based on your interests and skills (which is why the "Difficulty" fields aren't completely precise.)  **Please talk to us early in the formation of your proposal; this allows us to give you more useful feedback.**

**TODO: Check whether we have additional mentors whom we can list, confirm the suggested mentors.**

## Lightweight Client

Namecoin needs a light client, which can be used to register/manage names or get name values trustlessly but without the need to store the full blockchain. There are multiple possible ideas to tackle this, depending on your interests and likes. Some examples of how this might be done: 

* Port an existing lightweight Bitcoin client to Namecoin.  We prefer projects that aim to support the naming features of Namecoin, not just the currency features.  You may find [this list of Bitcoin wallets](https://bitcoin.org/en/choose-your-wallet) to be useful, but be aware that not all of the Bitcoin wallets listed there are suitable.  In particular, clients that are not fully open-source do not qualify; nor do clients that require a server component that is not fully open-source; nor do wallets that require a DRM-enforcing operating system such as Apple iOS; nor do wallets that require fully trusting a third party.  If you are porting a client that uses the BitcoinJ library, you may find [Ross Nicoll's libdohj library](https://github.com/dogecoin/libdohj) to be useful.
* Name database commitment system (for example, a Merkle tree system).  This could make it much more expensive for lookup services to lie about the state of names, make such lies more easily detectable, and allow lightweight clients to only download a subset of the name database.  This has some similarities to the UTXO set commitments that have been proposed in Bitcoin (and used in Electrum's protocol), but Namecoin's needs don't exactly match those of Bitcoin.  You may find [Daniel Kraft's proposal](https://forum.namecoin.org/viewtopic.php?f=5&t=2239) to be interesting.

Adding P2P wire protocol commands may be fine if it helps you in accomplishing this, as long as doing so doesn't cause our codebase to diverge too much from upstream Bitcoin.  Changing blockchain consensus rules is not off the table, but it would require convincing us that doing so has a clear benefit.  You are free to propose a codebase of your choice to work with as well as your own ideas for implementing a light client and discuss your choice with the community.  You may find the following writeups by Hugo Landau interesting, although take them with a grain of salt as they are somewhat outdated: [Namecoin Deployment Types](https://github.com/hlandau/ncdocs/blob/master/nctypes-intro.md) and [State of Namecoin](https://github.com/hlandau/ncdocs/blob/master/stateofnamecoin.md).

**Difficulty**: Medium-Hard

**Requirements**: If porting an existing Bitcoin client, a strong understanding of how Bitcoin and Namecoin work.  If working on a name database commitment system, familiarity with commitment systems such as those using Merkle trees.

**Expected Outcomes/Deliverables**: A light client that is usable and supports name transactions.

**Possible Mentors**: Daniel?, Ryan?, Jeremy, Joseph?, Brandon?, Hugo?

## Domain Name Integration with Additional Platforms/Applications

Namecoin currently supports integration with the DNS resolvers on desktop OS's such as GNU/Linux, Windows, and macOS.  Integration with the Tor anonymity network on desktop OS's (with domain names pointing to IP addresses, DNS names, and onion services) is almost ready for use.  However, there are several other applications and environments where Namecoin domain names would be beneficial.  Some of these are:

* The Android mobile OS.  This might also include porting an Android Light Client to Namecoin.
* The [I2P](https://geti2p.net/) anonymity network (names point to IP addresses, DNS names, and eepsites).
* The [Freenet](https://freenetproject.org/) anonymous hosting network (names point to freesites).
* The [ZeroNet](https://zeronet.io/) decentralized hosting network (names point to ZeroNet sites).  ZeroNet already has partial support for Namecoin but currently uses a centralized server to look up names.
* The IPFS decentralized hosting network (names point to IPFS sites).

You are free to integrate Namecoin domain names into any additional platform(s) or application(s) of your choice, as long as they do not enforce DRM (this means Apple iOS is not acceptable).  In many cases, it is likely that you would benefit from talking to the developers of whatever application you would be working with (e.g. if you're implementing Namecoin support in I2P, you might want to talk to the developers of an I2P client implementation).

**Difficulty**: Easy-Medium

**Requirements**: Familiarity with whatever platform(s) or application(s) you would be integrating Namecoin with.

**Expected Outcomes/Deliverables**: A working integration of Namecoin domain names with the platform(s) or application(s) of your choice.

**Possible Mentors**: Joseph?, Jeremy, Brandon?, Daniel?, Hugo?

## Identity Integration with Additional Applications

Namecoin could be used for identities with a variety of protocols and applications.  Some of these are:

* Website single sign-on.  Daniel Kraft's NameID is an interesting proof-of-concept of this, but it is questionable whether OpenID or a Firefox extension are the best ways to do this.  For example, TLS client certificate authentication has been suggested as a more standard alternative.
* OpenSSH client login.
* Bitmessage.  Daniel Kraft implemented a proof-of-concept that is now part of Bitmessage, but it could be significantly improved, e.g. by implementing reverse lookups (Bitmessage addresses to Namecoin identities).  Be careful about blockchain bloat; it might be more scalable to implement reverse lookups in the Bitmessage protocol rather than the Namecoin blockchain.
* OTR (Off The Record Messaging) and OMEMO.  Daniel Kraft implemented a proof-of-concept for Pidgin, but it is obsolete, and Pidgin itself is no longer considered particularly safe.  There are lots of IM clients that support OTR or OMEMO; adding Namecoin support to one of them would be beneficial.
* PGP.  Phelix implemented a proof-of-concept of a Namecoin PGP keyserver, but it is based on outdated code and might benefit from a fresh attempt.
* Ricochet.  **TODO: Check with Special about whether this is a good idea for 2018.**
* Tox.  **TODO: Check with Tox devs about whether this is a good idea for 2018**
* Ring.  **TODO: Check with Ring devs about whether this is a good idea for 2018**
* Cryptocurrency addresses.  May be relevant to the work by Netki and by [Monero's OpenAlias project](https://www.openalias.org/).

**Difficulty**: Easy-Medium

**Requirements**: Familiarity with whatever protocol(s) or application(s) you would be integrating Namecoin with.

**Expected Outcomes/Deliverables**: A working integration of Namecoin identities with the protocol(s) or application(s) of your choice.

**Possible Mentors**: Daniel?, Jeremy, maybe someone else????

## Name Management UI Improvements

Namecoin's name management UI isn't particularly friendly to novices; it expects them to be familiar with JSON and to be able to look up details in a fairly complicated specification.  Some kinds of poor security practice (e.g. using the same keys to update a name's value as can be used to transfer the name) may go unnoticed by average users, and other configuration errors may be difficult to debug by users unfamiliar with DNSSEC.  Some useful features of the Namecoin protocol remain unavailable in the UI, hampering their adoption.  You could help improve the UI in several ways:

* Provide a user-friendly UI for choosing values for the various JSON fields in the Namecoin domain names and identities specifications, which automatically constructs the JSON.
* When registering a domain name or identity, also register a throwaway name that is used to store the actual value of the name.  By automatically configuring the main name to import a value from the throwaway name, and defaulting to storing the main name and the throwaway name in different wallets, the main name can be kept offline while the throwaway name can be kept in an online wallet.  This reduces risk of name theft considerably.
* Check existing name configurations for errors (e.g. JSON errors or valid JSON that doesn't conform to the Namecoin domain names or identities specification), and inform the user of what's wrong.  Hugo Landau has written some code in ncdns that may be helpful for this.
* Namecoin's protocol supports atomic name trades, in which Alice can purchase a name from Bob without counterparty risk.  This could help reduce issues with squatted names.  Make this functionality available in the UI in a simple way.  Both interactive trades (where both parties select and confirm each other) and non-interactive trades (where a trade offer can be posted publicly and another user can accept the offer without further interaction) are possible; you can implement either (or both).  You may find [Phelix's ANTPY](https://github.com/phelixnmc/antpy) and [Ryan Castellucci's nametrade](https://github.com/ryancdotorg/nametrade) implementations to be interesting.
* Implement automatic renewal of names so that users don't accidentally let their name expire, and/or automatic pre-signing of renewals when the name is registered/updated so that the wallet doesn't need to be unlocked in order to renew.  Be careful about deanonymization vectors based on timing when a name is renewed!
* Allow users to control the transaction fees they pay for name operations.  (Right now the default fee policy is always used.)

You are free to implement any name management UI improvements of your choice in any Namecoin wallet that you choose.  (Since Namecoin Core is the only currently available wallet application, this would mean either implementing your UI improvements into Namecoin Core or combining some of these UI tasks with creating a Lightweight Client.)

**Difficulty**: Easy-Hard

**Requirements**: Familiarity with UI design in whatever language and UI framework your chosen wallet application uses.  (For example, Namecoin Core uses C++ with Qt.)

**Expected Outcomes/Deliverables**: Modified Namecoin wallet software with your UI improvements.

**Possible Mentors**: Brandon?, Joseph?, Daniel?, Ryan?, Hugo?, Jeremy?

## Anonymity

Namecoin suffers the same problems with traceability (lack of anonymity) of coin flows that Bitcoin does, plus additional linkability given the data attached to name transactions.  When Namecoin is used for its stated purpose to provide censorship-resistant browsing and registration of domains for possibly controversial purposes (e.g. under repressive regimes), this may be even a bigger problem than with Bitcoin.  You could improve this situation in several ways:

* Improve usability of registering a name anonymously by automating a purchase of namecoins using an anonymous currency like Monero or Zcash as part of the name registration process.  This reduces the risk of private data leaking while purchasing namecoins (e.g. banking details at an exchange, or Bitcoin blockchain analysis data).  Integrating with a decentralized exchange such as [Bitsquare](https://bitsquare.io/) might be a useful way to do this (this might also make it easier to register Namecoin names with national currencies).  Future advances in blockchain technology such as atomic cross-chain trades might be applicable, so try to make sure that your code could be reused with as few changes as possible for such use cases once they're ready.
* Keep each registered name separate on the blockchain.  This reduces the risk of private data leaking in the Namecoin blockchain.  This might be done using the Coin Control or HD Wallet features of wallet software.
* Improve the ability for Namecoin-related software to run safely when routed over Tor or other anonymity networks.  This might include testing for and fixing proxy leaks, application-layer protocol leaks, timing metadata leaks, and incorrect or missing stream isolation, among other things.  It's okay if this work includes some fixes to upstream software (e.g. Bitcoin wallet software), but a significant component of your project should be Namecoin-specific.

You are free to choose any anonymity improvements you wish in any Namecoin-related software of your choice.

**Difficulty**: Medium-Hard

**Requirements**: Familiarity with Bitcoin's anonymity properties/attacks.

**Expected Outcomes/Deliverables**: Modified Namecoin software with your anonymity improvements.

**Possible Mentors**: Jeremy, Daniel?, Ryan?, Joseph?, Brandon????, Hugo????

## Search Engine / Metrics

When we're asked how many people use Namecoin, what they use it for, and what example Namecoin websites exist, we currently have to say that we don't know.  This is a totally unnecessary conundrum; the blockchain is public and the websites are public.  You can help us learn more about how Namecoin is used in the real world by creating a search engine and metrics analysis system for Namecoin.  Since Namecoin is decentralized, the decentralized search engine YaCy would be a great fit for this.  This project idea has two components:

* Create scripts to automatically scan the Namecoin blockchain and index each domain name in YaCy.  This could be done via a standard DNS resolver running on the YaCy computer for clearnet websites; you may need to get creative if you wish to index websites hosted on networks like Tor, I2P, Freenet, ZeroNet, or IPFS.
* Create scripts to generate statistics about usage.  For example: How many websites are using Namecoin?  How many distinct pages exist on a Namecoin website?  (E.g. are they landing pages for squatted names or are they full websites?)  What record types are they using?  (IPv4, IPv6, Tor, I2P, etc.?)  Are they mirrors of non-Namecoin websites?  Do mirrors of non-Namecoin websites and the non-Namecoin websites that they're mirroring link to each other?  Use your imagination; what can we learn about how Namecoin domain names are used in the real world?

You are free to choose what types of addresses to crawl and what statistics to measure.

**Difficulty**: Easy-Medium

**Requirements**: Familiarity with YaCy.

**Expected Outcomes/Deliverables**: Scripts for populating a YaCy index of Namecoin websites and generating statistics about those websites.

**Possible Mentors**: Jeremy, maybe someone else????

## Your Own Namecoin Use or Enhancement

Namecoin can be used for lots of novel and interesting use cases, and existing use cases are open for improvement.  Propose your own use case or improvement, and discuss it with us!  Anecdotally, we've heard from other GSoC mentor organizations that the most successful projects were ones that a student came up with themselves rather than took from a suggestion list, so don't be afraid to be creative.  Also note that there's nothing wrong with working on more than one of the categories we've suggested.  For example, anonymous users often have bandwidth constraints, so combining Anonymity with a Lightweight Client might make sense; just be careful not to choose a bigger scope than you can expect to do over the summer.












# TODO: Are the below suggestions desirable?

## Compact encoding?

Namecoin currently uses JSON encoding for the values of names, which isn't particularly compact and wastes blockchain storage.  It may be useful to look into alternate encodings, including CBOR or the use of compression.  **TODO: Do we want this to be a GSoC suggestion?  Should it be combined with another idea (maybe Lightweight Client)?**

## Soft Hard Name Fork?

## Block explorer?

## Binary transparency?
