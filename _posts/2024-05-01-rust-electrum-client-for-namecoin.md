---
layout: post
title: "rust-electrum-client for Namecoin"
author: Jeremy Rand
tags: [News]
---

Recently, I needed to use [rust-electrum-client](https://github.com/bitcoindevkit/rust-electrum-client) with Namecoin. I was kind of worried that this would be nontrivial, but as it turned out, I was able to get it working in just a few days of hacking, despite this being my first foray into Rust.

The main incompatibility that came up was that Namecoin headers have an AuxPoW header after the 80-byte Bitcoin-style header, and the deserialization functions in rust-electrum-client really didn't like that trailing data. There were two deserialization functions that I had to patch to truncate the headers at 80 bytes, which got header parsing to work properly. My current patches are very unclean, and are not really suitable for publication, but I'm planning to clean them up and publish patches over the next few weeks.

Once this was done, a few non-code workarounds were additionally needed (more on these later!), and I was then able to use a real-world rust-electrum-client application with Namecoin. I'll talk more in a future post about what downstream code is using this.

This work was funded by Cyphrs.
