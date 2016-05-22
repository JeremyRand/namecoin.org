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

If you are only using Namecoin to look up names (e.g. browsing .bit domains), then you do not need to encrypt or back up your wallet.

### How much does it cost to register a domain (a.k.a. a name)? 
The cost includes a network fee and a transaction fee. As of 2014-8-01 the network fee is 0.01 NMC and the transaction fee is 0.005 NMC. There are plans to make this more dynamic.

### How do I obtain namecoins? Can i mine them?
You can mine them alongside bitcoins or trade them, see [How to get Namecoins](https://wiki.namecoin.info/index.php?title=How_to_get_Namecoins).

### Who gets the network fee? 
The network fees are destroyed by the transaction. Nobody gets them.

### Who gets the transaction fee? 
The miners do, just like in Bitcoin. The standard transaction fee is 0.005 NMC. You can increase this amount if you want to improve the chance that the transaction will be processed as soon as possible.

### How long are names good for? 
You have to execute an update on a name every 35,999 blocks at the latest (between 200 and 250 days), otherwise it expires. Usually there are no network fees for updates. There is a fee of 0.005 NMC, however, if you update very early.

### How do i browse a .bit domain?

There are a few different ways: Web proxys, nmcontrol, ncdns, changing your dns settings etc.

[How to browse .bit domains](https://wiki.namecoin.info/index.php?title=How_to_browse_.bit_domains)

If you have the zeronet software installed you can visit zeronet enabled .bit domains.

### How do i register and host a .bit domain?

[Register and Configure .bit Domains](https://wiki.namecoin.info/index.php?title=Register_and_Configure_.bit_Domains)

### Do I have to pay renewal fees? 
Not at the moment, although there are plans to change this..

### Can Namecoin be used to register other data types? 
Namecoin can be used to register any data type of data. Each record must follow the same rules (expire time, data size limit, etc). Before introducing a new namespace it is recommended to discuss it on the forum first. Note that for storing significant amounts of data there are other better suited options. If it absolutely has to be in a blockchain consider qora, blockstore, datacoin, and filecoin.

## Design 


### What is a namespace? 

Namespaces distinguish between different type of names in Namecoin.

Like "d/domain" for .bit domains or "id/name" for identity.

It depends on the software that reads the information from namecoin. A developer can use a different namespace for a different purpose. For example for "domain.meme" anyone could register the name "meme/domain" and if you had a "meme" TLD aware program it will work. Or if you have software capable of reading the blockchain you could hardcode it to check for updates by looking into the name "program/coolproyect" and then only the person controlling this name could dictate what is the latest version of your program, where to download the hash etc.

### Why is there a separate name_new step? 

This is to prevent others from stealing your new name by registering it quickly themselves when they see your transaction. The name is not broadcasted, only a salted hash of it. There is a mandatory 12 block wait that gives you enough time to broadcast your name with name_firstupdate, reducing the chance that someone will get in a name_firstupdate ahead of you.

### Why is there a network fee? 
The network fee is initially high, but will be negligible after a couple of years. It is used to slow down the initial registration rate so that plenty of desirable names are left for late adopters.

### How are names represented? 
Names and values are attached to special coins with a value of 0.01 NMC. Updates are performed by creating a transaction with the name's previous coin as input. Think of it like a colored coin.

### What if I spend that special coin by mistake? 
The code prevents those coins from being used for normal payments.

## Bitcoin vs. Namecoin

### What is the relationship of this project to Bitcoin?

The Namecoin codebase consists of the Bitcoin codebase with relatively minor changes (~400 lines) and addtional functionality built on top on it. The mining procedure is identical but the block chain is separate, thus creating Namecoin. This approach was taken because Bitcoin developers wanted to focus almost exclusively on making Bitcoin a viable *currency* while the Namecoin developers were interested in building a general data-value store. Bitcoin is a data-value store as well but it attempts to limit the type of data that can be stored to financial transactions, so it is more of a monetary-value store. Namecoin leverages Bitcoin's monetary-value store but focuses more on additional information which can be stored, such as a domain name system or an identification/authorization database.

### What is the difference to Bitcoin?

* There are additional commands for special transactions containing *names* and *data* (key/value pairs).
* The most important commands are: name_new, name_firstupdate and name_update.
* The coins used to pay for a name_firstupdate operation are being "destroyed", i.e. every new name reduces the finally usable maximum of 21 Million NMC by 0.01 NMC.
* `name_new`, `name_firstupdate` and `name_update` contain a pair of name/value which expires after 36,000 blocks (between 200 and 250 days).
* The `d/` prefix is used to register a domain name, without the .bit TLD: `{     "name" : "d/opennic",     "value" : "what you want",     "expires_in" : 10227 }`
* The `id/` prefix is used to register an identity, see http://nameid.org/
* Energy-efficient: if you are already mining bitcoins you can merge-mine Namecoins at no extra cost for hardware and electricity. Examples for merge-mining pools: mmpool.org, eligius.st, p2pool.org and many others.

### What are the similarities with Bitcoin?

* 21 millions namecoins total, minus the lost coins.
* 50 coins are generated each block, each 210000 blocks (around 4 years), the reward halves by two.
* Security: more than half of the Bitcoin miners also mine namecoin, giving it a staggering difficulty.
* Anonymous founder: Vinced, like Satoshi, never revealed his identity and dissapeared around the same time, leaving Namecoin project wild in the open, to flourish only thanks to the help of enthusiasts of the open source community.
* Open source platform: Anyone can improve the code and report issues on [Github](https://github.com/namecoin/namecoin) and even use it on other projects.

## Weaknesses 

### Name Stealing 
Name stealing on Namecoin is already much more difficult than with traditional domains.  However, once stolen, it's very difficult to recover a .bit domain.  We are working on reducing the damage that the thief can do and reducing the attack surface.  Overall, we feel that .bit is safer than traditional domains.

The simplest response to these concerns is to use the PhishTank. Any high-profile thefts or websites that engage in activity considered harmful to end users will end up in the PhishTank and similar client-side defensive mechanisms.   This removes much of the incentive to steal domains in the first place.

Secondly, we are working on removing the need for a private key from all maintenance tasks.  One can already use an 'import' statement to delegate name updates to a secondary name. There is also an "anyone pays" proposal that would allow for renewal of the domains using an arbitrary Namecoin address.  However, advanced users can already write renewal scripts on an offline, air gaped machine and upload them to the blockchain.  These techniques allow the private key for the primary name to be stowed away into cold storage unless the domain is sold.

We feel that such solutions make .bit domains **safer** than the traditional domains: Sex.com was famously stolen **with a fax** and it took *5 years and $4.5 million* in legal fees to recover.  The damages awarded to the original owner have yet to be paid.  While this case occurred in the 90s, we know of national and commercial registrars that could easily tricked into transferring a domain.

### 51% Attack 
The threat of the 51% attack is similar to that of Bitcoin, indeed Namecoin shares miners with Bitcoin.  Double spends are bad but miners have an economic interest in Namecoin's overall value.  Furthermore, it's not like an attacker can arbitrarily steal domains.  The shenanigans that can be carried out are all obvious and clients can be built to negate the impact of such actions. A 51% attack requires and adversary with tens of millions of dollars they want to blow on screwing with Namecoin and Bitcoin for a few weeks until they run out of money. This is highly unlikely and concerns about a 51% attack are mostly just [bike-shedding](https://en.wikipedia.org/wiki/Parkinson%27s_law_of_triviality).

See the [51% Attack](https://wiki.namecoin.info/index.php?title=51%25_Attack) article for a more thorough analysis.

### Squatting 
Squatting is simultaneously the most commonly cited and least important issue facing Namecoin.  While irritating in the short term, [it’s a solved problem](https://wiki.namecoin.info/index.php?title=Squatting).

### Blockchain Anonymity 

Like Bitcoin, Namecoin is not anonymous.

Only after Zerocash has been released and gone through some level of peer review it can be added to the code.  This will enable domain owners to escape political persecution.  This is a very real use-case, we have had political dissidents request this functionality but for the moment it's necessary to use a VPN or Tor to achieve complete anonymity.
