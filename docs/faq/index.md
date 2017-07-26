---
layout: page
title: FAQ
---

{::options parse_block_html="true" /}

## General

### How does Namecoin work? 

The Namecoin software is used to register names and store associated values in the blockchain, a shared database distributed by a P2P network in a secure way. The software can then be used to query the database and retrieve data.

### Do I need to back up my wallet?

If you're using Namecoin to register or otherwise own names, or to transfer namecoins, then you do need to periodically back up your wallet.  Like Bitcoin, your wallet's keys are located in your `wallet.dat` file.  You should encrypt this file by going to `Settings` > `Encrypt Wallet` and making a backup thereafter. Close the Namecoin client and make a backup of your `wallet.dat` file in your Namecoin profile folder.  (On GNU/Linux, this is usually `~/.namecoin/`; on Windows, it is usually `C:\Users\<Your Username>\AppData\Roaming\Namecoin\`).  It is currently recommended to back up more often than every 100 transactions (including both currency and name transactions).

If you are only using Namecoin to look up names (e.g. browsing `.bit` domains), then you do not need to encrypt or back up your wallet.

### How much does it cost to register a domain (a.k.a. a name)? 

The cost includes a registration fee and a transaction fee. The registration fee is 0.01 NMC, and the transaction fee is determined dynamically by miners (just like in Bitcoin).  The registration fee might be made dynamic in the future, to improve economic incentives.

### How do I obtain namecoins? Can I mine them?

You can mine them alongside bitcoins or trade them, see [How to get Namecoins](https://wiki.namecoin.org/index.php?title=How_to_get_Namecoins).

### Who gets the registration fee? 

The registration fees are destroyed by the transaction. Nobody gets them.

### Who gets the transaction fee? 

The miners do, just like in Bitcoin. Paying higher fees improves the chance that the transaction will be processed quickly.  Like Bitcoin, the client will suggest a fee that is likely to be processed quickly.

### How long are names good for? 

You have to renew or update a name every 35,999 blocks at the latest (between 200 and 250 days), otherwise it expires. There are no registration fees for renewals or updates, but a transaction fee does apply.

### How do I browse a .bit domain?

See [Browsing .bit Websites](https://bit.namecoin.org/browse.html).

You can also use [ncdns]({{site.baseurl}}docs/ncdns) (experimental).  If you have the [ZeroNet](https://zeronet.io/) software installed, you can visit ZeroNet-enabled .bit domains.

### How do I register and host a .bit domain?

[Register and Configure .bit Domains](https://wiki.namecoin.org/index.php?title=Register_and_Configure_.bit_Domains)

### Do I have to pay renewal fees? 

Other than the standard transaction fee, not at the moment.  This might change in the future, to improve economic incentives.

### What applications is Namecoin well-suited to?

Consider that Namecoin values are limited to 520 bytes, and that the block size limit is somewhere between 500 kB and 1 MB.  (It's lower than Bitcoin's.)  Given that blocks occur every 10 minutes on average (usually fluctuating between 1 and 60 minutes), and that names have to be updated or renewed at least every 35,999 blocks, try to figure out whether your application would comfortably fit into blocks if your application became widespread.  Domain names and identities are applications that are near the upper limit of the scale that Namecoin can handle.  For example, misusing the Namecoin blockchain as a decentralized file storage is not feasible.  There are several other decentralized systems that serve this purpose way more efficiently.  In many cases, if you want to store data that is larger than 520 bytes, or that is updated very often, you may prefer to only store a content hash or a public key in the blockchain, along with information on where to get the full data.  The full data can then be authenticated using Namecoin as a trust anchor without storing the entire data in Namecoin.  An example of this usage is the ability to delegate .bit domain names to an external DNSSEC nameserver, authenticated by a DS record in the blockchain.

If you're developing an application, consider doing your development on the Namecoin Testnet.  This prevents your testing from bloating the production blockchain, and also allows you to test without spending real money on names.  If more than one implementation might have the same use case, consider writing a spec so that incompatible implementations of similar ideas don't become a problem.

### What is the smallest currency unit of Namecoin called?

The smallest currency unit of Namecoin is called the *swartz* (similar to the *satoshi* in Bitcoin).  It is named after Aaron Swartz, the activist who was murdered by the U.S. government, and who [proposed Nakanames](https://web.archive.org/web/20170424134548/http://www.aaronsw.com/weblog/squarezooko) (which, along with BitDNS, described the concept that was later implemented as Namecoin).

## Design 

### What is a namespace? 

Namespaces are name prefixes used by applications to distinguish between different type of names in Namecoin.  For example, `d/example` is the domain name `example.bit`, and `id/example` is an identity.  Namespaces help prevent multiple applications from accidentally conflicting.  Namecoin itself isn't aware of namespaces, and namespaces don't have any effect on validation rules; they are only used by higher-level applications that use Namecoin.

### Why is there a separate `name_new` step? 

This is to prevent others from stealing your new name by registering it quickly themselves when they see your transaction. The name is not broadcasted during the `name_new` step, only a salted hash of it. There is a mandatory minimum delay of 12 blocks before you can broadcast your name with `name_firstupdate`; this means that by the time other people know what name you're registering, they would have to reverse at least 12 blocks in order to steal the name.

### How are names represented? 

Names and values are attached to special coins with a value of 0.01 NMC. Updates are performed by creating a transaction with the name's previous coin as input. Think of it like a colored coin.

### What if I spend that special coin by mistake? 

The code prevents those coins from being used for normal payments.

## Comparison of Namecoin to other projects

### What is the relationship of Namecoin to Bitcoin?

The Namecoin codebase consists of the Bitcoin codebase with relatively minor changes (~400 lines) and addtional functionality built on top on it. The mining procedure is identical but the block chain is separate, thus creating Namecoin. This approach was taken because Bitcoin developers wanted to focus almost exclusively on making Bitcoin a viable *currency* while the Namecoin developers were interested in building a *naming system*.  Because of the different intended use cases between the two projects, consensus and protocol rules might make sense in one but not ther other.  Examples of places where it could make sense to have different protocol or consensus rules:

* Namecoin's consensus rules need to enforce uniqueness of names.  While it is possible to store data in Bitcoin (e.g. key/value pairs in `OP_RETURN` outputs), uniqueness is not enforced by Bitcoin.  It would be theoretically possible to build a layer on top of Bitcoin that discards `OP_RETURN` outputs that don't respect uniqueness (e.g. a name operation that steals someone else's name), but any such layer would not be enforced by miners.  If transaction validity rules are not enforced by miners, then they are not backed by [PoW](https://bitcoin.org/en/glossary/proof-of-work), which means that [SPV-based lightweight clients](https://bitcoin.org/en/glossary/simplified-payment-verification) will fail to enforce those validity rules.
* Since consumers expect different fees for financial transactions versus name registrations, and since the volume of financial transactions worldwide versus name registrations worldwide are different, Namecoin and Bitcoin might have different optimal block sizes.
* In a currency, inflation attacks are fatal, while in a naming system, they simply amount to a spamming or squatting attack: bad, but not anywhere near fatal.  Therefore, decisions about features such as zk-SNARK-based anonymity (which introduce a risk of inflation attacks) might come to different conclusions between Namecoin and Bitcoin.
* Some script features that make sense for Namecoin might not make sense for Bitcoin, e.g. allowing a scriptPubKey to restrict the scriptPubKeys of any spending transaction.  In a naming system, features like this could make renewing and updating names more convenient and secure, but in a currency, they could be harmful to fungibility.
* Coinbase commitments to the name database could be enforced by Namecoin consensus rules, allowing SPV proofs of a name's nonexistence to be created.

In general, the Namecoin developers attempt to minimize our patchset against Bitcoin.  If a feature makes sense to have in Bitcoin, we try to get it into Bitcoin and then merge it to Namecoin; Namecoin usually only introduces differences from Bitcoin in cases where the proposed change wouldn't make sense for Bitcoin due to the differing use cases.  Although it is theoretically possible to use Namecoin as a general-purpose currency, the Namecoin developers do not encourage this use case.  There are lots of cryptocurrency projects out there that are specifically designed for such usage (e.g. Bitcoin); if you're looking for a currency, you should use one of those projects.

### What is the difference between Namecoin and Bitcoin?

* There are additional commands for special transactions containing *names* and *data* (key/value pairs).
* The most important commands are: `name_new`, `name_firstupdate`, and `name_update`.
* The coins used to pay for a `name_firstupdate` operation are destroyed, i.e. every new name reduces the finally usable maximum of 21 million NMC by 0.01 NMC.
* `name_new`, `name_firstupdate` and `name_update` contain a pair of name/value which expires after 36,000 blocks (between 200 and 250 days).
* The `d/` prefix is used to register a domain name, without the .bit TLD: `{     "name" : "d/opennic",     "value" : "what you want",     "expires_in" : 10227 }`
* The `id/` prefix is used to register an identity, see [NameID](https://nameid.org/).
* Energy-efficient: if you are already mining bitcoins you can merge-mine namecoins at no extra cost for hardware and electricity. Examples for merge-mining pools: mmpool.org, eligius.st, p2pool.org and many others.

### What are the similarities between Namecoin and Bitcoin?

* 21 million coins total, minus the lost coins.
* 50 coins are generated each block at the beginning; the reward halves each 210000 blocks (around 4 years).
* Security: a large fraction of Bitcoin miners also mine Namecoin, giving it a staggering difficulty.
* Pseudonymous founder: Vince, like Satoshi, never revealed his real-world identity and dissapeared around the same time, leaving Namecoin project wild in the open, to flourish only thanks to the help of enthusiasts in the FLOSS community.
* Free / libre / open-source platform: Anyone can improve the code and report issues on [GitHub](https://github.com/namecoin/) and even use it on other projects.

### How does Namecoin compare to Tor Onion Services?

The Tor Project's Onion Services (which have a `.onion` top-level domain) use domains which are a public key hash.  This means that their domain names are not human-meaningful, whereas Namecoin domain names are human-meaningful.  Namecoin's `.bit` domains can point to `.onion` domains, providing a human-meaningful naming layer on top of Tor Onion Services.  Blockchain-based systems like Namecoin are, at this time, unable to match the cryptographic security guarantees (against impersonation or deanonymization attacks) that systems like Onion Service names provide when used directly, but Namecoin's human-meaningful names do make Namecoin more resistant than Onion Service names to some classes of attacks that exploit human psychology rather than breaking cryptography.  For example, humans have trouble remembering a public key hash or recognizing a public key hash as the correct one; this is much better with meaningful names such as Namecoin names (or ICANN DNS names).  Attackers can exploit this property of Onion Service names in order to trick users into visiting the incorrect website.  We believe that both systems serve a useful purpose, and determining whether direct usage of Onion Service names or Namecoin naming for Onion Services is more secure for a given user requires consideration of that user's threat model.

### How does Namecoin compare to Blockstack?

Below is a comparison table of Namecoin and Blockstack (with Bitcoin added for reference).

|  | **Namecoin** | **Blockstack** | **Bitcoin** |
---|--------------|----------------|-------------|
| **Lightweight validation mode** | [SPV](https://bitcoin.org/en/glossary/simplified-payment-verification) backed by [PoW](https://bitcoin.org/en/glossary/proof-of-work) (e.g. BitcoinJ+libdohj). | Checkpoints provided by a trusted 3rd party. Blockstack refers to this as "Consensus Hashes", "SNV" ("Simplified Name Verification"), or (confusingly) "SPV". Is not backed by PoW and has no relation to Bitcoin's SPV threat model. | SPV backed by PoW (e.g. BitcoinJ). | 
| **Hashrate attesting to transaction ordering** | ~45% of Bitcoin as of 2016 Aug 23. | 100% of Bitcoin. | 100% of Bitcoin. | 
| **Hashrate attesting to transaction validity** | ~45% of Bitcoin as of 2016 Aug 23. | 0% of Bitcoin (miners do not attest to transaction validity). | 100% of Bitcoin. | 
| **Miners possessing a majority of hashrate** | None. | None. | None. | 
| **Mining pools influencing a majority of hashrate** | None. | None. | None. | 
| **Legal jurisdictions influencing pools with a majority of hashrate** | China. | China. | China. | 
| **Infrastructure capable of censoring all new blocks (non-selectively)** | Bitcoin Relay Network, by censoring all Bitcoin blocks that commit to Namecoin blocks. | Bitcoin Relay Network. | Bitcoin Relay Network. | 
| **Infrastructure capable of censoring new blocks based on content (e.g. targeting a name)** | None. | Bitcoin Relay Network. | Bitcoin Relay Network. | 
| **Blockchain download size (for full node)** | 2.52 GB as of 2016 Aug 23 ([source](https://bchain.info/NMC/)). | 75.57 GB as of 2016 Aug 23 ([source](https://bchain.info/BTC/)) (same as Bitcoin). | 75.57 GB as of 2016 Aug 23 ([source](https://bchain.info/BTC/)). | 
| **Consensus codebase** | Fork of Bitcoin Core with minimal changes (primarily merged mining and 3 new opcodes for altering a name database). | Completely new codebase. | Bitcoin Core. | 
| **Data storage** | Choice between blockchain (similar threat model to Bitcoin) and external storage (lower transaction fees, higher scalability).  External DNS storage currently supports nameservers (threat model is DNSSEC with user-supplied keys; the DNSSEC keys have similar threat model to Bitcoin).  External identity storage currently supports PGP keyservers. | Required external storage; by default [DHT](https://en.wikipedia.org/wiki/Distributed_hash_table).  See the opinions of Bitcoin developers [Peter Todd](https://bitcointalk.org/index.php?topic=395761.msg5970778;topicseen#msg5970778) and [Greg Maxwell](https://bitcointalk.org/index.php?topic=662734.msg7521013;topicseen#msg7521013) about DHT's. | Blockchain. | 
| **Funding sources and ethics** | Crowdfunding, donations, consulting/contracting (e.g. for F2Pool and an employee of Kraken).  Has refused funding opportunities that were perceived to create conflicts of interest regarding user freedom, privacy, and security.  Frequently references WikiLeaks and Ed Snowden in specification examples and other development discussion. | Funded by an investor who has [endorsed cryptographic backdoors](https://web.archive.org/web/20160319061046/http://avc.com/2016/03/privacy-absolutism/) and who considers [ROT13](https://en.wikipedia.org/wiki/ROT13) to be a ["serious" and "intriguing" security mechanism](https://twitter.com/csoghoian/status/709908777038954496). | (N/A) | 
| **Do the developers run services that hold users' private keys?** | No.  Many years ago, a former Namecoin developer did run such a service.  It has been discontinued, as the current Namecoin developer team considers such services to be harmful and a liability. | [Yes, Onename.](https://onename.com/) | Not as far as we know. | 

### How does Namecoin compare to Monero?

Monero's MoneroDNS project is similar in concept to Namecoin.  MoneroDNS's technical differences to Namecoin are similar to Monero's technical differences to Bitcoin.  Monero has had much less technical review than Bitcoin, and merge-mined chains based on Monero have significantly less hashrate security available to them than merge-mined chains based on Bitcoin.  On the other hand, Monero's small size enables them to liberally experiment with more advanced features and cryptography, whereas Bitcoin-based systems like Namecoin are more conservative.  The Namecoin and Monero development teams are cooperating on areas of common interest, as both projects agree that Namecoin and Monero both have a future.

## Weaknesses 

### How easy is it for names to be stolen?  What can be done if it happens?

For an attacker who does not have a majority of hashrate, stealing a Namecoin name is, roughly speaking, equivalent to the task of stealing bitcoins.  This usually requires stealing the private key which owns the name.  Assuming that proper security measures are in place by the owner, this is very difficult.  However, if a user fails to keep their private keys safe, all bets are off.  The standard method for attempting to steal bitcoins is to use malware; this is likely to be equally effective for stealing Namecoin names.  Users can protect themselves using all the standard methods of avoiding malware, which are out of scope of this FAQ.

The good news is that the script system inherent in Bitcoin and Namecoin is designed to enable features that make theft more difficult.  Many features are under development that would allow users considerable flexibility in constructing anti-theft policies that meet their needs.  For example:

* **Multisig** (similar to Bitcoin) would allow names to be controlled by M-of-N keys.  Some of these keys could belong to the various directors of a company, be stored in a secure location, or be stored by semi-trusted service providers.  This is currently supported by the Namecoin protocol and consensus rules, but not well-exposed to end users.
* **Offline signing** (similar to Bitcoin) would allow names to be controlled by keys that are located on an air-gapped computer, an isolated offline Qubes virtual machine, or a hardware wallet.  This is currently supported by the Namecoin protocol and consensus rules, but not well-exposed to end users.
* **Delegated renewal** (Namecoin-specific) would allow a key to be authorized to renew a name, but not change its value or its owner.  Efforts are underway to add this to the Namecoin protocol and consensus rules.
* **Delegated alteration** (Namecoin-specific) would allow a key to be authorized to alter the value of a name, but not change its owner.  This is supported, but not well exposed to end users. Further improvements are underway.
* **Delegated partial alteration** (Namecoin-specific) would allow a key to be authorized to alter a specific subset of the value of a name (for example, be allowed to change a domain name's IP address but not its TLS certificate), but not change other parts of the value or its owner.  This is supported, but not well-exposed to end users.  Further improvements are underway.

The above features can, of course, be combined arbitrarily for additional layered security.

Unfortunately, if all of the above security measures fail (or are not in use for a given name), and a name does get stolen, it is very difficult to recover it.  Legal action might be able to fine or imprison the thief if they refuse to return the name, but this is not reliable, given that there is no guarantee that the thief will be identifiable, or that the thief will be in a legal jurisdiction who cares.  Furthermore, since names do get sold or transferred on a regular basis, it would be difficult to prove that the name was not voluntarily transferred.  (False claims of theft are problematic in Bitcoin too.)  In cases where it is obvious that a theft has occurred (e.g. a previously reputable website starts serving malware), voluntary and user-bypassable third-party blacklists (e.g. [PhishTank](https://en.wikipedia.org/wiki/PhishTank)) could be reasonably effective at protecting users in some circumstances.  While this doesn't recover the name, it does reduce the incentive to attempt to steal names.

We are unaware of convincing empirical evidence of how Namecoin's theft risk compares to that of the ICANN domain name system when the recommended security procedures of both are in use; this is difficult to measure because it is likely that a significant number of Namecoin users and ICANN domain name system users are not using the recommended security procedures.

### What is the threat posed by 51% attacks?

Information about what a 51% attacker can do in Bitcoin is [described on the Bitcoin StackExchange](https://bitcoin.stackexchange.com/a/662).  Namecoin is quite similar.  The primary things that adversely affect Namecoin are reversing transactions sent by the attacker and preventing transactions from gaining confirmations.

Reversing transactions sent by the attacker would allow name registrations to be stolen if the reversed transaction is a `name_firstupdate`.  This is because prior to being registered, names are considered to be "anyone can spend", meaning that prior to the registration, any arbitrary attacker is equally in ownership of a name as the user who actually registers it.  Preventing transactions from gaining any confirmations would allow names to be stolen if all transactions for a name are prevented from confirming until the name expires after 36000 blocks, at which point the attacker can register it.

Both of these attacks are detectable.  In the case of reversing transactions, the evidence would be an extremely long fork in the blockchain, possibly thousands of blocks long or longer.  In the case of preventing transactions from confirming, the evidence would be that the blockchain indicates that a name expired and was re-registered.  In both cases, it is detectable which names were attacked.  In the case of preventing transactions from confirming, it is also possible for the legitimate owner of the stolen name to register a new name after the attack is over, and sign it with the owner key of the original name, thus proving common ownership and allowing secure resurrection of the name.  The only way to prevent this resurrection is for the attacker to continue to expend mining resources on the attack for as long as they with to prevent the name from being resurrected.  In the case of reversing transactions, it is not possible to prove ownership of the original name and resurrect it.  Luckily, reversing old transactions is considerably more expensive than preventing new transactions from confirming.

It is noteworthy that a 51% attacker cannot sell a name to a user and then steal back the name.  Nor can a 51% attacker buy a name from a seller and then steal back the money.  This is because Namecoin supports *atomic* name trades: reversing the purchase payment also reverses the name transfer, and vice versa.  Double-spending of `name_update` transactions also isn't beneficial to an attacker, because `name_update` transactions typically are sent by a user to themself, meaning that the attacker could only scam themself.

In both Bitcoin and Namecoin, the Chinese government has jurisdiction over a majority of hashpower.  This is problematic for both Bitcoin and Namecoin, and should be fixed in both.  Because not all Bitcoin miners also mine Namecoin, F2Pool previously had a majority of Namecoin hashpower (they no longer do).  This was also problematic when it was the case.  However, in practice, the Chinese government has considerably more motivation to perform a 51% attack than F2Pool does.  (The Chinese government has a [history of messing with Internet traffic](https://en.wikipedia.org/wiki/Internet_censorship_in_China).  F2Pool has supported Namecoin development both financially and logistically, which makes it unlikely that they would want to attack it.)

A majority of Bitcoin's hashpower is routed via the Bitcoin Relay Network, which has the ability to censor Bitcoin blocks that pass through it.  This produces incentives for Bitcoin miners to self-censor any blocks that might violate any policy introduced in the future by Bitcoin Relay Network, because routing blocks through Bitcoin Relay Network reduces orphan rates for miners.  Namecoin's blocks are much smaller than Bitcoin's, and therefore Namecoin does not have similar incentives for centralized block relay infrastructure.  While it is possible for Bitcoin Relay Network to attack Namecoin by censoring Bitcoin blocks that commit to merge-mined Namecoin blocks, it is not feasible for Bitcoin Relay Network to look inside the Namecoin blocks that are committed to, which means that Bitcoin Relay Network cannot censor Namecoin blocks by content as they can with Bitcoin blocks.  Bitcoin Relay Network is operated by Bitcoin Core developer Matt Corallo, who is unlikely to want to attack Bitcoin (just as F2Pool is unlikely to want to attack Namecoin).

The takeaway here is that while F2Pool theoretically used to be capable of attacking Namecoin (but not Bitcoin), and Bitcoin Relay Network is theoretically capable of attacking Bitcoin (but not Namecoin), *in practice* the party with the most motivation to attack either chain (the Chinese government) has jurisdiction over a hashrate majority of both Bitcoin and Namecoin.  Mining decentralization is an active research area, and we hope that significant improvements in this area are made, as they would improve the security of both Bitcoin and Namecoin.

### Is squatting a problem?  What can be done about it?

There are several types of squatting concerns sometimes raised in relation to Namecoin.

The first concern is that too many potentially high-value domains, e.g. `d/google`, have been squatted for the purpose of resale.  This is not a problem that can be solved in a decentralized system, because "squatting on `d/google`" is defined as "owning `d/google` while not being the real-world company named Google", and determining that a given name is or is not owned by a given real-world entity requires some trusted party.  Raising the price of names wouldn't have any effect on this, because no matter what the cost of registering a name is, the resale value of `d/google` is likely to be higher.

The second concern is that too many potentially high-value domains have been squatted for the purpose of impersonation.  This is not a problem specific to Namecoin; phishing sites exist in the ICANN world too, and are frequently countered by using systems such as web-of-trust and voluntary user-bypassable third-party blacklists (e.g [PhishTank](https://en.wikipedia.org/wiki/PhishTank)).  There is no reason to think that similar counters would not work in Namecoin.

The third concern is that single entities can squat on a large number of names, which introduces centralization into the space of squatted names.  For comparison, ICANN domain names are squatted a lot, but the space of squatted names is very decentralized, which reduces abusive behavior such as would happen if most squatted names belonged to one of a few people.  This concern could be resolved by raising the price of name registrations, so that a squatter with a given investment budget cannot register as many names without selling or otherwise using them to recoup costs.  While raising prices sounds like a great plan, the devil is in the details: increasing prices constitutes a softfork, and decreasing prices constitutes a hardfork.  Since cryptocurrencies like Namecoin have an exchange rate that varies over time, the optimal name price might need regular adjustment.  There is ongoing research into how regular name price adjustment could be done safely and non-disruptively, and research in the wider cryptocurrency world on block size adjustment (which is a similar problem in many ways) may be applicable.

At the moment, the current developers consider other issues to be somewhat higher priority.  For example, getting a domain name without dealing with squatters doesn't mean much if it's difficult for people to view your website.  Once development of other areas has progressed further, we do intend to spend a larger fraction of our time on improving name pricing.  However, if new developers want to get involved with proposing, prototyping, or analyzing name price systems, we would be delighted to have the assistance.

In the meantime, practical advice is that if you want a name but it's squatted, try to contact the owner (many squatters leave contact information in the value of their names) and see if they'll let you have it.  We have heard of many cases where squatters either gave away names or sold them for very little money if the recipient actually planned to use the name rather than resell it.  If they demand money that you're unwilling to pay, consider registering a different name.  It's unlikely that the website or service you want to set up can only work with that one specific name.  Strategies for finding an unused ICANN domain name or an untrademarked business name are likely to be applicable for Namecoin too.

### Is Namecoin anonymous?

Like Bitcoin, Namecoin is not anonymous.  A thorough description of Bitcoin's poor anonymity properties is outside the scope of this FAQ.

When used properly in conjunction with Tor, Namecoin *may* offer sufficient pseudonymity or location-anonymity for many use cases.  Users who need these properties are advised to carefully evaluate their specific situation.  Using Namecoin over Tor does *not* by itself magically make you anonymous.

We recognize that better anonymity is an important use case.  We occasionally receive questions from users about whether Namecoin can be used anonymously.  While we don't know much about these users (for obvious reasons), some of them appear to be in circumstances where failure of anonymity could lead to significant negative consequences.  We aim to support these use cases in the future, but right now it would be irresponsible and reckless to do so.

We are currently engaging with projects that provide blockchain anonymity (e.g. Monero and Zcash), with the goal of achieving similar anonymity for Namecoin.  Both Monero and Zcash have mathematical security proofs of their anonymity, subject to given assumptions and a given anonymity set.  Blockchain anonymity is also an active research area, so further innovations may very well occur in the future.
