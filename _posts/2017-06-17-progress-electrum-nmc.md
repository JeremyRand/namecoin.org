---
layout: post
title: Progress on Electrum-NMC
author: ahmedbodi
tags: [News]
---

Work on the electrum port for Namecoin has been moving along nicely. It was decided that we will use the electrum-client from spesmilo, along with the electrumX server. ElectrumX was chosen due to the original electrum-server being discontinued a few months ago. 
So far the electrum client has been ported over for compatability with electrumX. This includes the re-branding, blockchain parameters and other electrum related settings for blockchain verification

On the roadmap now are:

* Extend electrumX NMC Support to allow for full veritification of AuxPow
* Modify new electrum client to verify the new AuxPow
* Add Name handling support to electrum

These repo's are for testing purposes only. Do not use these unless your willing to risk losing funds. 

[Client](https://github.com/Multicoin-co/electrum-nmc)    
[Server/ Not uploaded yet](https://github.com/Multicoin-co/electrum-nmc-server)
