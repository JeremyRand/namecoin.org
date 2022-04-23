---
layout: post
title: "Go DNS Seeder Improvements"
author: Jeremy Rand
tags: [News]
---

Any P2P network has to deal with initial peer discovery.  Bitcoin and Namecoin mostly solve this via *DNS seeds*: special domain names that return a large number of IP addresses corresponding to Bitcoin/Namecoin nodes.  If you've used Namecoin Core, you've probably encountered the dreaded "no peers" symptom.  This is because, unfortunately, the primary DNS seeder implementation used by Bitcoin (by Pieter Wuille) is neither Freedom Software (it's under an All Rights Reserved license, so Namecoin cannot legally use it) nor memory-safe (it's in C++).  To help improve Namecoin peer discovery (and maybe Bitcoin too), I've submitted the following improvements to Lyndsay Roger's Go-based DNS seeder (which is both Freedom Software and memory-safe):

* Run without root privileges via `setcap`.
    * This is a good alternative to `iptables`, which was already supported.
* Use multiple initial IP's.
    * Helpful if there are no operational DNS seeds when you start a crawl.
* Receive multiple peers per crawl.
    * Fixes a Bitcoin protocol implementation bug that caused crawls to stall.
* Support Bitcoin Core's `seeds.txt` API.
    * Allows us to export the seed list into Namecoin Core, so that even if the DNS seed is offline, Namecoin Core can still find peers.
* Route TCP traffic over SOCKS5 proxy.
    * Prerequisite for crawling onion-service peers.

Lyndsay has merged all of these except for the SOCKS5 patch, which is waiting on some review from the Tor developers.  Huge thanks to Lyndsay for reviewing and merging the patches -- I'm hoping that these patches will result in much better peer discovery for Namecoin Core (which has always been a pain point for us).
