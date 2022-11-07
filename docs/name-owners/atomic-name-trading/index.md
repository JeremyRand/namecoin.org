---
layout: page
title: Atomic Name Trading
---

{::options parse_block_html="true" /}

Because both currency transactions and name operations share a common technical backing (the Namecoin blockchain), it is possible to construct transactions that transfer both namecoins and a name *atomically*. This means that it is guaranteed that either both transfers succeed or both transfers are not executed at all. A potentially very useful application of this technique is trading of names for namecoins, since this makes it possible to sell or buy a name in a completely trustless manner without the necessity to use escrow.

There are two protocols for atomic name trading:

* The newer protocol is supported only by Electrum-NMC v4.0.6+; the older protocol is supported only by Namecoin Core.
* The newer protocol is accessible via both CLI/RPC and GUI; the older protocol is CLI/RPC-only.
* The newer protocol is *non-interactive*: one party can publish an offer (e.g. on a pastebin or forum), and the second party can then accept the offer without needing to send information back to the first party.  Under the hood, this is achieved via Bitcoin smart contract techniques such as `SIGHASH` and `nLockTime`.  The older protocol is *interactive*: the parties must coordinate the trade in a multi-step negotiation procedure.
* The newer protocol supports auctions (CLI/RPC only); the older protocol does not.
* Both protocols utilize Layer 2 (the only step that hits the blockchain is the final step that completes the trade).  This avoids excessive fees and blockchain bloat.
* Both protocols are fully decentralized and trustless, modulo the reliance on a publishing mechanism (e.g. a pastebin or forum), which could be implemented in a decentralized, censorship-resistant manner but is considered out of scope.  A malicious pastebin operator can censor offers but cannot tamper with them.
* It probably goes without saying that the older protocol will be scrapped in the future, once the newer protocol is available in Namecoin Core.

The basic workflow is as follows:

## Electrum-NMC: Bob Prepares a Buy Offer

We will assume as an example that Bob wants to buy `d/my-cool-domain` from Sally for 10 NMC.  Bob can create a Buy Offer in Electrum-NMC like this:

Console:

```
name_buy("d/my-cool-domain", amount="10")
```

CLI:

```
electrum-nmc name_buy d/my-cool-domain 10
```

GUI:

Use the Buy Names tab.

The command will return a Buy Offer in hex format.  Bob can give this Buy Offer to Sally via any medium, such as email or a public pastebin.

Note that by creating this Buy Offer, Bob is committing to pay 10 NMC to anyone who transfers `d/my-cool-domain` to him, even if Sally no longer controls that name at the time of the trade.  Bob can revoke this obligation by double-spending the currency input used by the Buy Offer.

## Electrum-NMC: Sally Accepts a Buy Offer

After Sally receives the Buy Offer from Bob, she can accept it in Electrum-NMC like this (she enters the Buy Offer in the `offer` parameter):

Console:

```
name_sell("d/my-cool-domain", requested_amount="10", offer="0123456789ABCDEF")
```

CLI:

```
electrum-nmc name_sell d/my-cool-domain 10 --offer 0123456789ABCDEF
```

GUI:

Use the Manage Names tab.

Note that the transaction fee to the miner will be paid by Sally.  The command will return a trade transaction.  Sally can now broadcast it in Electrum-NMC like this (she enters the trade transaction as the argument to `broadcast`):

Console:

```
broadcast("FEDCBA9876543210")
```

CLI:

```
electrum-nmc broadcast FEDCBA9876543210
```

GUI:

Broadcasted automatically from the Manage Names tab.

This completes the trade; once the transaction is confirmed, Bob now owns `d/my-cool-domain` and Sally is 10 NMC (minus the transaction fee) richer.

## Electrum-NMC: Sally Prepares a Sell Offer

We will assume as an example that Sally wants to sell `d/my-cool-domain` to anyone willing to pay 10 NMC for it.  Sally can create a Sell Offer in Electrum-NMC like this:

Console:

```
name_sell("d/my-cool-domain", requested_amount="10")
```

CLI:

```
electrum-nmc name_sell d/my-cool-domain 10
```

GUI:

Use the Manage Names tab.

The command will return a Sell Offer in hex format.  Sally can publish this Sell Offer via any public medium, such as a pastebin.

Note that by creating this Sell Offer, Sally is committing to transfer `d/my-cool-domain` to anyone who pays her 10 NMC; she does not have any control over who accepts the Sell Offer.  Sally can revoke this obligation by updating or renewing `d/my-cool-domain`.  If Sally specifically wants only Bob to be able to receive `d/my-cool-domain`, then she should ask Bob to create a Buy Offer instead.

## Electrum-NMC: Bob Accepts a Sell Offer

After Bob finds Sally's Sell Offer on a public medium such as a pastebin, he can accept it in Electrum-NMC like this (he enters the Sell Offer in the `offer` parameter):

Console:

```
name_buy("d/my-cool-domain", amount="10", offer="0123456789ABCDEF")
```

CLI:

```
electrum-nmc name_buy d/my-cool-domain 10 --offer 0123456789ABCDEF
```

GUI:

Use the Buy Names tab.

Note that the transaction fee to the miner will be paid by Bob.  The command will return a trade transaction.  Bob can now broadcast it in Electrum-NMC like this (he enters the trade transaction as the argument to `broadcast`):

Console:

```
broadcast("FEDCBA9876543210")
```

CLI:

```
electrum-nmc broadcast FEDCBA9876543210
```

GUI:

Broadcasted automatically from the Buy Names tab.

This completes the trade; once the transaction is confirmed, Bob now owns `d/my-cool-domain` and Sally is 10 NMC richer.

## Electrum-NMC: Sally Prepares an Auction

We will assume as an example that Sally wants to sell `d/my-cool-domain` via a [Dutch auction](https://en.wikipedia.org/wiki/Dutch_auction).  Sally has chosen to open the auction at block height 300k with price 10 NMC, and intends to decrease the price by 1 NMC every 10 blocks until it reaches the minimum price of 1 NMC.  Sally can create an Auction in Electrum-NMC like this:

Console:

```
name_sell_auction("d/my-cool-domain", requested_amounts=["10","9","8","7","6","5","4","3","2","1"], locktimes=[300000,300010,300020,300030,300040,300050,300060,300070,300080,300090])
```

CLI:

```
electrum-nmc name_sell_auction d/my-cool-domain '["10","9","8","7","6","5","4","3","2","1"]' '[300000,300010,300020,300030,300040,300050,300060,300070,300080,300090]'
```

Creating Auctions is not available in the GUI yet.

The command will return an Auction in JSON format.  Sally can publish this Auction via any public medium, such as a pastebin.

Note that by creating this Auction, Sally is committing to transfer `d/my-cool-domain` to the first person who bids on the auction; she does not have any control over who bids.  Sally can revoke this obligation by updating or renewing `d/my-cool-domain`.

## Electrum-NMC: Bob Bids on an Auction

After Bob finds Sally's Auction on a public medium such as a pastebin, he can bid on it in Electrum-NMC like this (he enters the Auction in the `offers` parameter):

Console:

```
name_buy_auction("d/my-cool-domain", amount="5", offers=["0123456789ABCDEF",...])
```

CLI:

```
electrum-nmc name_buy_auction d/my-cool-domain 5 '["0123456789ABCDEF",...]'
```

Bidding on Auctions is not available in the GUI yet.

Note that the transaction fee to the miner will be paid by Bob.  The command will return a bid transaction corresponding to the Auction stage selected by the following algorithm:

1. Disqualify any stages that are above Bob's bid price.
2. Prefer stages that can be broadcasted earlier.
3. If more than one stage can be broadcasted immediately, prefer the stage with the lowest price.

Bob can now broadcast his bid in Electrum-NMC like this (he enters the bid transaction as the argument to `broadcast`):

Console:

```
broadcast("FEDCBA9876543210")
```

CLI:

```
electrum-nmc broadcast FEDCBA9876543210
```

This completes the Auction; once the transaction is confirmed, Bob now owns `d/my-cool-domain` and Sally is 5 NMC richer.

## Namecoin Core: Bob Prepares a Buy Offer

We will assume as an example that Bob wants to buy `d/my-cool-domain` from Sally for 10 NMC. The work-flow for this transaction is shown below on the testnet.

Bob finds the domain of interest and Sally's offer to sell the domain. This could happen, for instance, via Sally using a special value for the domain or by some other channel out of the scope of this page. For instance: 

```
$ namecoin-cli name_show d/my-cool-domain
{
  "name" : "d/my-cool-domain",
  "value" : "for sale by Sally for 10 NMC, pay to mm1mGfLEMjFPyGsbkcoYiN6yiPcBztK6Dw",
  "txid" : "00abd8282558574369038722d24c65355e1b98ca3f00d7d09cb7bea8732b598a",
  "address" : "mt2vTvKfvfb9TKfxB8ns7NVrY4cYzRs73b",
  "expires_in" : 36000
}
```

Now, Bob can prepare an atomic transaction that sends 10 NMC to Sally and transfers `d/my-cool-domain` at the same time to himself. We assume that Sally wants the 10 NMC sent to `mm1mGfLEMjFPyGsbkcoYiN6yiPcBztK6Dw`, while Bob wants the name to be transferred to `mwKDtLZ9EHSeL8TjkZAcppsnb7M3LRhoGR`. 

To construct the transaction, Bob needs to use the `createrawtransaction` RPC command. It needs three pieces of information: 

* Currency inputs: These will be provided by Bob to pay the 10 NMC to Sally as well as the transaction fee.
* Currency outputs: 10 NMC to Sally's address and potential change back to Bob.
* A `name_update` operation to be performed.

In this example, these three parts will be written in seperate files for simplicity, and then these files will be used in the final RPC command. Of course, it is also possible to write everything directly onto the command line. 

## Namecoin Core: Currency Inputs

Assuming a transaction fee of 0.005 NMC is appropriate, Bob needs to find one of his unspent inputs of at least 10.005 NMC to include in the transaction. For this, the `listunspent` command can be used. A suitable input could be this one: 

```
$ namecoin-cli listunspent
[
  {
    "txid" : "12ed9f83b83e248e2ca19e0197933914013145981fcacc5ba47c27e60eacbfaa",
    "vout" : 0,
    "scriptPubKey" : "41040a1b1f8d3b497c7ee13e441539b9e1fe5b4ea96696a4827a74e02aa6c4df4383209de42bbc02ce90e28e5e109b9b1b7dcfd14b02449603c4320faf394719a8f1ac",
    "amount" : 974.90000000,
    "confirmations" : 2
  }
]
```

With a size of 974.9 NMC, it is large enough. It will produce `974.9 - 10.005 = 964.895 NMC` change. The interesting data fields for our purpose are `txid` and `vout`. They will form the currency input for `createrawtransaction`, so Bob writes into the file `tx.in`: 

```
[{
  "txid": "12ed9f83b83e248e2ca19e0197933914013145981fcacc5ba47c27e60eacbfaa",
  "vout": 0
}]
```

If no single unspent output is large enough, Bob could instead either include multiple inputs into this file, or produce a suitable output first by sending a sufficiently large amount to himself.

## Namecoin Core: Currency Outputs

Required currency outputs are the price paid to Sally as well as the change paid back to Bob. So Bob first creates a new address of his own to receive the change. We will use `mwAEUoA3NzSmASyWxrTK1mhsd4pFvruoXj` for this purpose. Then, the corresponding data to be put in `tx.out` is: 

```
{
  "mm1mGfLEMjFPyGsbkcoYiN6yiPcBztK6Dw": 10.0,
  "mwAEUoA3NzSmASyWxrTK1mhsd4pFvruoXj": 964.895
}
```

## Namecoin Core: The Name Operation

The last part of the transaction that needs to be created is the name operation. It should be a `name_update` of `d/my-cool-domain` to some value (can be chosen by Bob already at this stage, but it usually isn't too important for a name trade) and to Bob's address `mwKDtLZ9EHSeL8TjkZAcppsnb7M3LRhoGR`. The corresponding data to put in the last file `tx.op` is this: 

```
{
  "op": "name_update",
  "name": "d/my-cool-domain",
  "value": "thanks from Sally",
  "address": "mwKDtLZ9EHSeL8TjkZAcppsnb7M3LRhoGR"
}
```

## Namecoin Core: Finishing the Offer

With all this in place, the transaction can be created: 

```
$ namecoin-cli createrawtransaction "$(cat tx.in)" "$(cat tx.out)" "$(cat tx.op)"
0071000002aabfac0ee6277ca45bccca1f9845310114399397019ea12c8e243eb8839fed120000000000ffffffff8a592b73a8beb79cd0d7003fca981b5e35654cd2228703694357582528d8ab000100000000ffffffff0300ca9a3b000000001976a9143c4b7d4b93bc6194087bbbc422fd6cd1a40f820e88ac60ed3877160000001976a914ab96c9bfb7c55de825be7f12bd38dd25b62be02b88ac40420f00000000003f5310642f6d792d636f6f6c2d646f6d61696e117468616e6b732066726f6d2053616c6c796d7576a914ad4a0929e9c7c95910534b93ec0727058a27f2b988ac00000000
```

Now Bob can check that everything is fine: 

<pre markdown="span">$ namecoin-cli decoderawtransaction <large hex string>
{
  "txid" : "88180e13968cea5ea611fcb34d26b5391bdc9118f8a35fad2495b07be9e88c7b",
  "version" : 28928,
  "locktime" : 0,
  "vin" : [
    {
      "txid" : "12ed9f83b83e248e2ca19e0197933914013145981fcacc5ba47c27e60eacbfaa",
      "vout" : 0,
      "scriptSig" : {
        "asm" : "",
        "hex" : ""
      },
      "value" : <strong>974.90000000</strong>,
      "sequence" : 4294967295
    },
    {
      "txid" : "00abd8282558574369038722d24c65355e1b98ca3f00d7d09cb7bea8732b598a",
      "vout" : 1,
      "scriptSig" : {
        "asm" : "",
        "hex" : ""
      },
      "value" : 0.01000000,
      "sequence" : 4294967295
    }
  ],
  "vout" : [
    {
      "value" : <strong>10.00000000</strong>,
      "n" : 0,
      "scriptPubKey" : {
        "asm" : "OP_DUP OP_HASH160 3c4b7d4b93bc6194087bbbc422fd6cd1a40f820e OP_EQUALVERIFY OP_CHECKSIG",
        "hex" : "76a9143c4b7d4b93bc6194087bbbc422fd6cd1a40f820e88ac",
        "reqSigs" : 1,
        "type" : "pubkeyhash",
        "addresses" : [
          "<strong>mm1mGfLEMjFPyGsbkcoYiN6yiPcBztK6Dw</strong>"
        ]
      }
    },
    {
      "value" : <strong>964.89500000</strong>,
      "n" : 1,
      "scriptPubKey" : {
        "asm" : "OP_DUP OP_HASH160 ab96c9bfb7c55de825be7f12bd38dd25b62be02b OP_EQUALVERIFY OP_CHECKSIG",
        "hex" : "76a914ab96c9bfb7c55de825be7f12bd38dd25b62be02b88ac",
        "reqSigs" : 1,
        "type" : "pubkeyhash",
        "addresses" : [
          "<strong>mwAEUoA3NzSmASyWxrTK1mhsd4pFvruoXj</strong>"
        ]
      }
    },
    {
      "value" : 0.01000000,
      "n" : 2,
      "scriptPubKey" : {
        "nameOp" : {
          "op" : "name_update",
          "name" : "d/my-cool-domain",
          "value" : "thanks from Sally"
        },
        "asm" : "NAME_OPERATION OP_DUP OP_HASH160 ad4a0929e9c7c95910534b93ec0727058a27f2b9 OP_EQUALVERIFY OP_CHECKSIG",
        "hex" : "5310642f6d792d636f6f6c2d646f6d61696e117468616e6b732066726f6d2053616c6c796d7576a914ad4a0929e9c7c95910534b93ec0727058a27f2b988ac",
        "reqSigs" : 1,
        "type" : "pubkeyhash",
        "addresses" : [
          "<strong>mwKDtLZ9EHSeL8TjkZAcppsnb7M3LRhoGR</strong>"
        ]
      }
    }
  ],
  "fees" : <strong>0.00500000</strong>
}</pre>

The most important pieces to check are highlighted above, but they are obviously the addresses as well as the amounts to be sent. It is also a good idea to check the calculated fees to make sure there's no typo in the amounts resulting in exorbitantly high fees.

When everything looks fine, Bob can sign his inputs to the transaction (after unlocking the wallet if it is password-protected): 

```
$ namecoin-cli signrawtransactionwithwallet <large hex string>
{
  "hex" : "0071000002aabfac0ee6277ca45bccca1f9845310114399397019ea12c8e243eb8839fed12000000004948304502202131c98c98a3dfe19713ec0e48917a502063028766abf292b46fb8e8a96d80df022100f22198c86e814eff3a2562a36de5249a425c08080cb174eace7da9529e6e16b801ffffffff8a592b73a8beb79cd0d7003fca981b5e35654cd2228703694357582528d8ab000100000000ffffffff0300ca9a3b000000001976a9143c4b7d4b93bc6194087bbbc422fd6cd1a40f820e88ac60ed3877160000001976a914ab96c9bfb7c55de825be7f12bd38dd25b62be02b88ac40420f00000000003f5310642f6d792d636f6f6c2d646f6d61696e117468616e6b732066726f6d2053616c6c796d7576a914ad4a0929e9c7c95910534b93ec0727058a27f2b988ac00000000",
  "complete" : false
}
```

Finally, Bob can now again check the signed transaction with `decoderawtransaction`, although this is not strictly necessary. He can now pass the partially-signed transaction (the hex string above) to Sally. 

## Namecoin Core: Sally Accepts the Offer

After Sally receives the partial transaction from Bob, she can check that everything is fine: 

```
$ namecoin-cli decoderawtransaction <large hex string>
{
  "txid" : "92a50d2495d66cbfb13ae87deaec059ab605a7bf737b10935f9e0f80065debac",
  "version" : 28928,
  "locktime" : 0,
  "vin" : [
    {
      "txid" : "12ed9f83b83e248e2ca19e0197933914013145981fcacc5ba47c27e60eacbfaa",
      "vout" : 0,
      "scriptSig" : {
        "asm" : "304502202131c98c98a3dfe19713ec0e48917a502063028766abf292b46fb8e8a96d80df022100f22198c86e814eff3a2562a36de5249a425c08080cb174eace7da9529e6e16b801",
        "hex" : "48304502202131c98c98a3dfe19713ec0e48917a502063028766abf292b46fb8e8a96d80df022100f22198c86e814eff3a2562a36de5249a425c08080cb174eace7da9529e6e16b801"
      },
      "value" : 974.90000000,
      "sequence" : 4294967295
    },
    {
      "txid" : "00abd8282558574369038722d24c65355e1b98ca3f00d7d09cb7bea8732b598a",
      "vout" : 1,
      "scriptSig" : {
        "asm" : "",
        "hex" : ""
      },
      "value" : 0.01000000,
      "sequence" : 4294967295
    }
  ],
  "vout" : [
    {
      "value" : 10.00000000,
      "n" : 0,
      "scriptPubKey" : {
        "asm" : "OP_DUP OP_HASH160 3c4b7d4b93bc6194087bbbc422fd6cd1a40f820e OP_EQUALVERIFY OP_CHECKSIG",
        "hex" : "76a9143c4b7d4b93bc6194087bbbc422fd6cd1a40f820e88ac",
        "reqSigs" : 1,
        "type" : "pubkeyhash",
        "addresses" : [
          "mm1mGfLEMjFPyGsbkcoYiN6yiPcBztK6Dw"
        ]
      }
    },
    {
      "value" : 964.89500000,
      "n" : 1,
      "scriptPubKey" : {
        "asm" : "OP_DUP OP_HASH160 ab96c9bfb7c55de825be7f12bd38dd25b62be02b OP_EQUALVERIFY OP_CHECKSIG",
        "hex" : "76a914ab96c9bfb7c55de825be7f12bd38dd25b62be02b88ac",
        "reqSigs" : 1,
        "type" : "pubkeyhash",
        "addresses" : [
          "mwAEUoA3NzSmASyWxrTK1mhsd4pFvruoXj"
        ]
      }
    },
    {
      "value" : 0.01000000,
      "n" : 2,
      "scriptPubKey" : {
        "nameOp" : {
          "op" : "name_update",
          "name" : "d/my-cool-domain",
          "value" : "thanks from Sally"
        },
        "asm" : "NAME_OPERATION OP_DUP OP_HASH160 ad4a0929e9c7c95910534b93ec0727058a27f2b9 OP_EQUALVERIFY OP_CHECKSIG",
        "hex" : "5310642f6d792d636f6f6c2d646f6d61696e117468616e6b732066726f6d2053616c6c796d7576a914ad4a0929e9c7c95910534b93ec0727058a27f2b988ac",
        "reqSigs" : 1,
        "type" : "pubkeyhash",
        "addresses" : [
          "mwKDtLZ9EHSeL8TjkZAcppsnb7M3LRhoGR"
        ]
      }
    }
  ],
  "fees" : 0.00500000
}
```

It is also crucial to check which inputs Bob has prepared for her to sign, so that she doesn't end up sending something she doesn't intend to send. I'm not completely sure what would be the easiest and safest way to ensure that the transaction from Bob is not malicious. A possible way to guard against this could be to look up the address holding the sold name, export its private key, and use `signrawtransactionwithwallet` with this private key as optional argument.

When everything is fine, Sally can sign her parts of the input: 

```
$ namecoin-cli signrawtransactionwithwallet <large hex string>
{
  "hex" : "0071000002aabfac0ee6277ca45bccca1f9845310114399397019ea12c8e243eb8839fed12000000004948304502206114a73cb307e17ec0d4c5d7caa9d87a2136da511a94b7bd8027aab726b2e67a022100bcebf433a9fa13d50c602f0cf54e1e7ab2ac851ca1ed9017ecf95d7dc02e25e401ffffffff8a592b73a8beb79cd0d7003fca981b5e35654cd2228703694357582528d8ab00010000008a47304402202620285799e7314e3da0573fe473354b6bf0eb29640254f073a124fe1454dc9d022019e4884853a5aba76059ff2bd23e6e5b6e8c607df0e46e83cbfc2ea26e67da4a01410467d20dd8d4c08674a13ea14540f09d62389c7d96bed805f5bbbb4079210dbf2ede9eeced9ab638a8d6ed01a909b91473e78a1ff65ba3de19b38203cf22c9d12dffffffff0300ca9a3b000000001976a9143c4b7d4b93bc6194087bbbc422fd6cd1a40f820e88ac60ed3877160000001976a914ab96c9bfb7c55de825be7f12bd38dd25b62be02b88ac40420f00000000003f5310642f6d792d636f6f6c2d646f6d61696e117468616e6b732066726f6d2053616c6c796d7576a914ad4a0929e9c7c95910534b93ec0727058a27f2b988ac00000000",
  "complete" : true
}
```

The transaction is now completely signed (when `"complete": true` is returned). Sally can check it again, and if everything is fine, she can relay it to the network: 

```
$ namecoin-cli sendrawtransaction <large hex string>
bdf33963b3635d41d597e07805ee8f0f8e637e7e09e46ff5117931e7ffdb8550
```

That's it. When the transaction is confirmed, the trade is completed. However, there's no risk to either Bob or Sally that the trade could ever be done only partially, i.e., someone cheat the other. 

## Credits

* Daniel Kraft: Initial interactive protocol.
* Phelix: Refined interactive UI.
* Ryan Castellucci: Non-interactive protocol.
* Yanmaani: Auction protocol.
* Jeremy Rand: Refined non-interactive UI.
