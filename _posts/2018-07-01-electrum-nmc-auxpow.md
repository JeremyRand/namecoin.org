---
layout: post
title: "Electrum-NMC: AuxPoW Deserialization"
author: Jeremy Rand
tags: [News]
---

Namecoin's merged mining, which allows miners to simultaneously mine a parent chain (usually Bitcoin) and any number of child chains (e.g. Namecoin and Huntercoin), is made possible by AuxPoW (auxilliary proof of work).  AuxPoW is a clever trick (first proposed by Bitcoin founder Satoshi Nakamoto, and first implemented by Namecoin founder Vincent Durham) that allows a block in the parent chain to commit to a block in any number of child chains, such that the child block can reference the parent block's PoW and thereby prove that the PoW committed to both the parent and child block.  AuxPoW doesn't impose any changes for the parent chain's consensus rules, but it does constitute a hardfork for the child chain (even for lightweight SPV clients of the child chain).  As a result, making Electrum-NMC validate PoW properly requires patching Electrum to support AuxPoW.

Recently I started hacking on Electrum-NMC to support AuxPoW.  This is not an entirely new problem; a previous Electrum fork ([Electrum-DOGE](https://github.com/electrumalt/electrum-doge)) already tried to implement AuxPoW.  Unfortunately, Electrum-DOGE's code quality is not exactly up to my standards, and besides that, it's a fork of a 3.5-year-old version of Electrum.  Electrum has evolved substantially since then, to the point that a straightforward merge isn't possible.  Additionally, it's not clear how many people have actually audited Electrum-DOGE for correctness.  That said, Electrum-DOGE's implementation is definitely a useful reference for determining how to do AuxPoW in Electrum.

Upon adding some debug output to Electrum-NMC, I observed that the first error that showed up was in deserializing block headers.  This makes sense, since in Bitcoin, all block headers are exactly 80 bytes, whereas in Namecoin, the 80 bytes are optionally followed by a variable-length AuxPoW header, which includes things such as the parent block's header, the parent block's coinbase transaction (variable length), and two Merkle branches (also variable length).  Electrum-DOGE's code for deserializing block headers wasn't directly mergeable, but it definitely was sufficient reference material to implement AuxPoW header deserialization in Electrum-NMC.

The next error that showed up was related to deserializing **chunks** of block headers.  Electrum groups block headers into chunks, where each chunk corresponds to a difficulty period (2016 block headers).  Electrum was, of course, assuming in the chunking code that a chunk was exactly `2016 * 80` bytes, which wasn't going to work with AuxPoW.  Fixing this was straightforward enough that I did so without using Electrum-DOGE as a reference (the chunking code has evolved enough in the last 3.5 years that using Electrum-DOGE as a reference would probably have taken more time than reimplementing from scratch).

The next step is dealing with serialization/deserialization of block headers to/from disk.  Naturally, Electrum's block header storage format assumes 80-byte headers, so fixing that will take some work.

There's also a licensing side effect of using Electrum-DOGE as a reference.  Electrum's license used to be GPLv3+, but since then they've relicensed to MIT.  Electrum-DOGE was forked from Electrum before the license change, and Electrum-DOGE's authors never relicensed.  As a result, the code I wrote that's based on Electrum-DOGE's codebase is a derivative work of GPLv3+-licensed code.  All of the code from upstream Electrum, as well as all of Namecoin's changes to Electrum and Electrum-DOGE, are still MIT-licensed, but the full combined work that constitutes Electrum-NMC is GPLv3-licensed.  This isn't really a huge problem (GPLv3+ is a perfectly fine free software license, and I wasn't intending to submit any of the AuxPoW code upstream anyway), but it's definitely noteworthy.

More Electrum AuxPoW work will be covered in future posts.

This work was funded by Cyphrs.
