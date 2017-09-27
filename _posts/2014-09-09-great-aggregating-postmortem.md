---
layout: post
title: The Great Aggregating Postmortem
author: Namecoin Developers
tags: [News]
redirect_from:
  - /post/97092159695/
  - /post/97092159695/the-great-aggregating-postmortem/
---
In the past couple of weeks, Namecoin suffered outages. Someone (whom we’ve nicknamed “The Aggregator”) tried consolidating a large quantity of “loose change” into a single address. When done correctly, this is good because it reduces the amount of data lite clients [Jeremy edit 2017 05 10: this means UTXO-pruned clients] need to keep. Unfortunately, a combination of volume and an oversight in fee policies led to miners getting knocked offline.

Within 48/hours, we released a patch fixing the low fees which should have stopped the transactions, but miners were slow to uptake the patch. We then increased performance and miners have since finished processing the remaining transactions.

Our initial response was slow because our core developers were traveling.  However, we are taking further steps to mitigate similar problems, improve our response times, and improve communications with miners.

Technical deep dive after the jump.

## Deep Dive

An unknown party (The Aggregator) began consolidating a massive volume (50,000-100,000K) of unspent outputs into a single address. However, namecoind has extreme difficulty selecting outputs to spend in a wallet containing tens of thousands of unspent outputs. The Aggregator appears to have attempted to address this by writing a script which manually built transactions spending 50 or 100 of those unspent outputs to [a new address (N8h1WYaCpnZrN72Mnassymhdrm7tT6q5yL)](https://namecha.in/address/N8h1WYaCpnZrN72Mnassymhdrm7tT6q5yL).

The problem is that each of these transactions were 17-30 KB and rather than the standard transaction fee of 0.005NMC per KB (rounded up), a flat fee of 0.005NMC was used. Namecoin, like Bitcoin actually has two fee values, `MIN_TX_FEE` (used by miners) and `MIN_RELAY_TX_FEE` (used by all other full nodes). This enables `MIN_TX_FEE` to be lowered down to `MIN_RELAY_TX_FEE` without nodes who haven’t yet upgraded refusing to forward transactions with the new lower fee. On Bitcoin, the ratio between `MIN_TX_FEE` and `MIN_RELAY_TX_FEE` is 5:1, but due to an oversight it was 50:1 in Namecoin. The result was that The Aggregator’s transactions had a large enough fee to be forwarded throughout the network, but were considered “insufficient fee/free” transactions by miners. Since there is very limited space in blocks for such transactions, they just kept building up in the miners’ memory pools. The volume of transactions soon began causing the merge-mining software run by pools to time-out trying to get work.

We released an initial “band-aid” patch which simply upped the `MIN_RELAY_TX_FEE` to stop these particular problem transactions from being rebroadcast or accepted by pools. Phelix and Domob put together an RC branch which fixed the performance issues associated with The Aggregator’s transactions. The transactions have all been process but we have had positive feedback from miners and will be merging the RC branch with mainline shortly.

[Additional techniques](https://groups.google.com/forum/?_escaped_fragment_=topic/namecoin/8RdXGNq3oSs#!topic/namecoin/8RdXGNq3oSs) are being discussed to ensure Namecoin can better handle situations where there are too many transactions being broadcast to process them all. One is to temporarily drop lowest-priority transactions in the event of a flood, while granting higher priority to name transactions (`NAME_NEW`, `NAME_FIRSTUPDATE`, `NAME_UPDATE`) in order to ensure that time-sensitive operations are not delayed. Bitcoin developers have planned enhancements which will further mitigate the problem.

It was observed that at one point only one pool was functional. This pool mined many blocks in a row with no transactions. We have identified this pool and have been in contact with the operator. They modified their software some time ago to ignore all transactions to address some performance issues. We do not believe there was any malicious intent on their part, and they have committed to working with us to get to a point where they can include transactions in their blocks.

## Response

Downtime is very bad; it opens us up to attacks and messes things up for businesses which depend on Namecoin. Trying to [guarantee some arbitrary level of uptime is a fool’s errand](https://www.joelonsoftware.com/2008/01/22/five-whys/) but we endeavour to outline what went wrong and what steps we are taking to prevent similar issues from occurring in the future.

Each step of our response was unacceptably slow: discovering a problem, diagnosing the problem, submitting patches, and coordinating with miners all took far longer than they needed to. Our response was primarily slowed by the fact that two of our lead developers (Ryan and Domob) were both away from home with limited or no availability. There were other issues as well, which are addressed in the following recommendations, but this single factor significantly slowed every step of the process. Had they been available, we would have had patches out within 24 hours of the initial problem.

### Initial Error Detection

The initial response times lagged mainly because we were unable to diagnose the problem. Individual developers (Ryan in particular) have patched clients which aid in diagnosing abnormalities. Initial detection of the problem lagged as well, individual users and developers noted problems but this took some time to reverberate. Automated monitoring system that will alert us to abnormalities on the network would be useful as well.

### Recommendations

* Document and share patched clients
    - Ryan
    - Anon developer M?
* Setup automated monitoring √
    - Block discovery times. √ Namecha.in has agreed to help.
    - Mining Pool ratios. √ Namecha.in + Ryan
* Testnet-in-a-box. (Domob/Ryan need to publish)

### Pushing Fixes

We went nearly 48 hours between having a bandaid patch and pushing out a statically compiled binary. This occurred for the following reasons:

* Indolering didn’t know how to properly communicate and distribute the fix.
* Limited communications with pool operators: it appears that the miners did not apply the initial bandaid patch which would have fixed network issues immediately.
* Even after the correct course of action was identified, it took 3-4 hours to create an official fork, produce a statically compiled binary, and post to the blog. We have been focusing our efforts on automating the build process for Libcoin, it should not be a problem moving forward.

### Recommendations

* Codify process for creating an emergency branch and binary √
* Setup new alerts mailing list and ask pool operators to join √
* Gather contact info for major pool operators. √
* Setup multi-user access to Twitter.
* Streamline build processes.
    - Jeremy Rand’s progress on Libcoin:
        + Travis CI/automated builds √
        + Functional unit testing √
        + Miner-specific unit tests (Libcoin doesn’t yet support mining).

### Readiness & Coordination

In first-aid, the first thing you do is order bystanders to contact emergency services, direct traffic, etc. More developers would obviously help, but, akin to first-aid, emergencies need someone ensuring that progress is being made. Without a proper first responder, you have a lot of bystanders simply standing around.

### Recommendations

* Fix developer mailing list. √
* Create security mailing list. (in progress)
* Appointing an official emergency coordinator to ensure things are moving.
    * Has contact information of developers (including phone numbers) and miners.
    * Alternate for when emergency contact is unavailable.
* Improve volunteer onboarding. (planned for September)
    * Improve documentation.
    * Better labeling of repos (deprecate mainline, rebrand Libcoind, etc)
    * Public development roadmap.
    * Organize bounties/tickets on needed items.

## Thanks

Big thanks to Domob, Ryan, Phelix, Jeremy Rand, Indolering, our anonymous devs, mining pool operators, and others for their help

While Domob and Phelix coded the final fix, many people contributed to development. Phelix and Jeremy Rand were omnipresent during the entire response, from raising the initial warning to diagnosing the problem, proposing fixes, and testing. At one point, Jeremy Rand pulled 36 hours with no sleep. Ryan was especially generous with his time during a business trip, pulling all-nighters to help us diagnose the problem, develop fixes, and communicate with the mining pools.

Indolering worked hard to coordinate the response and communicate with miners. Our anonymous developers were a **huge** help, it’s a shame we can’t thank them publicly.

Luke-Jr, CloudHashing, BitMinter, GHash.io, and MegaBigPower helped us test our patches and provided feedback on miner performance.
