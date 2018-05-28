---
layout: post
title: "Electrum-NMC: Code Review and Rebase"
author: Jeremy Rand
tags: [News]
---

Last year, Ahmed [posted about his progress]({{site.baseurl}}2017/06/17/progress-electrum-nmc.html) porting Electrum to Namecoin.  Electrum-NMC has been on the back burner for me lately, due to the TLS and BitcoinJ efforts taking up most of my time.  However, today I found time to inspect Ahmed's branch.

Three main things were on my agenda:

1. Code review of the existing changes.  Generally I don't like to move code to the GitHub Namecoin organization unless I've actually reviewed it for sanity.  I made a few tweaks and bugfixes to Ahmed's code, but for the most part the code review went smoothly.
2. Rebase against current master branch of Electrum.  This actually went surprisingly well, given that Ahmed's branch is about 11 months old.  The vast majority of the merge conflicts were due to the `electrum` package being renamed to `electrum-nmc`, which causes unfortunate merge conflicts every time upstream messes with the `import` statements at the top of a Python file.  Unfortunately I don't know of any way around this, and a cursory check of altcoin Electrum ports shows that they do the same thing, so I guess we're going to live with it.  The good news is that those types of merge conflicts are very easy to manually resolve.
3. Additional rebranding beyond what Ahmed's branch does.  In particular, I swapped out the ElectrumX Bitcoin server addresses and replaced them with ElectrumX Namecoin server addresses.  (Right now there's only 1 public ElectrumX Namecoin server.  We need more of them --- if you'd like to help us, please consider starting up an ElectrumX Namecoin server and sending a PR to ElectrumX that adds your server to the public list.)  I also swapped out the Bitcoin block explorers and replaced them with Namecoin explorers.  (I also gave the Namecoin explorers names that include scary warnings for the subset of explorers that are wiretapped by CloudFlare, discriminate against Tor users, don't support names, or are non-libre.)

The resulting code [is now on GitHub](https://github.com/namecoin/electrum-nmc).  I've successfully sent some coins from Namecoin Core to Electrum-NMC and back without any difficulty.

Regarding next steps, I'll defer to Ahmed's post:

> On the roadmap now are:
> 
> * Extend electrumX NMC Support to allow for full veritification of AuxPow
> * Modify new electrum client to verify the new AuxPow
> * Add Name handling support to electrum

Hopefully these won't be incredibly difficult.  I might post binaries of the current codebase before I try to tackle these (but note that I'm not familiar with the Electrum packaging scripts yet, so there's a good chance that I'll break something and/or find something that I broke earlier).

This work was funded by NLnet Foundation's Internet Hardening Fund.
