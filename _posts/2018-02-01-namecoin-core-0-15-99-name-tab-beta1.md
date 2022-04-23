---
layout: post
title: "Namecoin Core 0.15.99-name-tab-beta1 Released"
author: Jeremy Rand
tags: [Releases, Namecoin Core Releases]
---

Namecoin Core 0.15.99-name-tab-beta1 has been released on the [Beta Downloads page]({{site.baseurl}}download/betas/#namecoin-core).  Changes since 0.13.99-name-tab-beta1:

* New features:
    + GUI
        - Significant rewrite of name GUI.  (Patch by brandonrobertz.)  **In particular, please torture-test the following**:
            * Full flow for registering names.
            * Full flow for updating and renewing names.
            * State display in the names list.
            * The above with mainnet, testnet, and regtest networks.
            * The above with encrypted locked, encrypted unlocked, and unencrypted wallets.
    + RPC
        - Remove name operation from `createrawtransaction` RPC method; add `namerawtransaction` RPC method.  This paves the way for various future improvements to the name GUI, including coin control, anonymity, fee control, registration without unlocking the wallet twice, and decreased transaction size.  You'll need to update your scripts if you currently use the raw transaction API for name transactions.  (Reported by JeremyRand, patch by domob1812.)
        - Restore `getblocktemplate` RPC method.  This improves workflow for software used by Bitcoin mining pools.  (Reported by DrHaribo, patch by domob1812.)
        - Add `createauxblock` and `submitauxblock` RPC methods.  This improves workflow for software used by Bitcoin mining pools.  (Patch by bitkevin.)
* Fixes:
    + GUI
        - Fix pending name registration bug, where the GUI requests a wallet unlock over and over and then errors with name registered.  (Patch by brandonrobertz.)
        - Fix bug where names weren't showing up in the Manage Names list properly until client had been restarted.  (Patch by brandonrobertz.)
    + P2P
        - Update seed nodes.  This should decrease likelihood of getting stuck without peers.  (Patch by JeremyRand.)
    + RPC
        - Fix crash when user attempts to broadcast an invalid raw transaction containing multiple `name_update` outputs.  (Reported by maxweisspoker, patch by domob1812.)
* Improvements from upstream Bitcoin Core.

Unfortunately, Windows and macOS builds are broken in this release, so only GNU/Linux binaries are available.  We expect Windows and macOS builds to be restored for the 0.15.99-name-tab-beta2 release, which is coming soon.
