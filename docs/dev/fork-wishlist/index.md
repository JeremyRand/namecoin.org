---
layout: page
title: Fork Wishlist
---

{::options parse_block_html="true" /}

This is a list of proposed consensus forks for Namecoin.  Inclusion on this list does not indicate widespread support of the Namecoin community.  This page only covers blockchain consensus rules; changes to namespace specs are not covered here.

## Hardforks

* [Require AuxPoW to be always active](https://github.com/namecoin/namecoin-core/pull/75).  This is a proposed solution to an incompatibility between BIP 9 (VersionBits) and AuxPoW.
* [AuxPoW: use BIP 122 chain ID](https://github.com/namecoin/namecoin-core/pull/75#issuecomment-228597157).
* AuxPoW: Use SegWit commitments.
* AuxPoW: "correct the merged-mining chain_id awkwardness" ([as midnightmagic put it]({{ site.forum_url }}/viewtopic.php?f=5&t=1824))
* AuxPoW: Use protocol from P2Pool.  Would decrease size of AuxPoW headers.  Implementation is in use already by P2Pool; spec incomplete.
* AuxPoW: Use protocol from Luke-Jr.  Would probably fix GBT, decrease size of AuxPoW headers more than P2Pool, requires a Bitcoin hardfork as well, implementation and spec not public yet.
* Disable BDB lock limit.
* Remove requirement that name operations appear in a transaction with the Namecoin tx nVersion.
* Increase limit on name value size.
* Change name fee structure (only a hardfork if it allows name outputs to hold less than 0.01 NMC).
* Multiple names per transaction.
* Extend name expiration period to circa 1 year.
* Make name expiration period use BIP 113 timestamps instead of block height.
* Store names as (Hash(name), Enc(name, value)) rather than (name, value).

## Softforks

* [AuxPoW: restrict timestamp difference to parent block]({{ site.forum_url }}/viewtopic.php?f=8&t=2455&hilit=softfork&sid=e59d68ac212f89a82ed1ccefc7da4996).
* Coinbase commitment for UNO set.
* Make dust spam outputs from the attack between blocks 39k and 41k unspendable.
* Segregated Name Values.
* Change name fee structure (only a softfork if it doesn't allow name outputs to hold less than 0.01 NMC).
* Delegated renewal.
