---
layout: page
title: Electrum-NMC
---

{::options parse_block_html="true" /}

Electrum-NMC is the Namecoin port of the lightweight Bitcoin wallet Electrum.

## Installation

You can download Electrum-NMC at the [Beta Downloads]({{ "/download/betas/#electrum-nmc" | relative_url }}) page.

## Name Management

Name transactions are visible in the History tab:

![Screenshot.]({{ "/images/screenshots/electrum-nmc/Names-in-History-Tab.png" | relative_url }})

Name operations are visible in the Outputs section of the Transaction Details dialog:

![Screenshot.]({{ "/images/screenshots/electrum-nmc/Name-in-Transaction-Details.png" | relative_url }})

You can enter a name you'd like to register in the Buy Names tab:

![Screenshot.]({{ "/images/screenshots/electrum-nmc/Buy-Name-Entry.png" | relative_url }})

If the name is available to register, you'll see a message like this:

![Domain test1.bit is available to register!]({{ "/images/screenshots/electrum-nmc/Buy-Name-Available.png" | relative_url }})

If the name is already taken, you'll instead see a message like this:

![Domain domob.bit is already registered, sorry!]({{ "/images/screenshots/electrum-nmc/Buy-Name-Taken.png" | relative_url }})

You can view a list of your registered names in the Manage Names tab:

![Screenshot.]({{ "/images/screenshots/electrum-nmc/Manage-Names-Tab.png" | relative_url }})

In-progress name registrations will show an ETA:

![Screenshot.]({{ "/images/screenshots/electrum-nmc/Registration-ETA.png" | relative_url }})

You can update or transfer a name by clicking "Configure Name...":

![Screenshot.]({{ "/images/screenshots/electrum-nmc/Configure-Name-Dialog.png" | relative_url }})

For domain names, the DNS Builder facilitates updating the name's data without manually touching JSON:

![Screenshot.]({{ "/images/screenshots/electrum-nmc/DNS-Builder.png" | relative_url }})

## Name Lookups

The `name_show` command can be called from the console:

![Screenshot.]({{ "/images/screenshots/electrum-nmc/Name-Show-in-Console-Tab.png" | relative_url }})

To integrate Electrum-NMC's name lookups with applications such as [ncdns]({{ "/docs/ncdns/" | relative_url }}), follow these instructions:

1. Enable Electrum-NMC's JSON-RPC interface, as per [the instructions from upstream Electrum](https://electrum.readthedocs.io/en/latest/merchant.html#jsonrpc-interface).
1. To test the RPC server, try running the following from a terminal (substitute your username, password, and port accordingly): `curl --data-binary '{"id":"curltext","method":"name_show","params":["d/nf"]}' http://username:password@127.0.0.1:7777`.  You should get something like this:
   ~~~
   {"result": {"name": "d/nf", "name_encoding": "ascii", "value": "{\"ip\":\"94.23.252.190\",\"map\":{\"_tcp\":{\"map\":{\"_443\":{\"tls\":[{\"d8\":[1,\"MFkwEwYHKoZIzj0CAQYIKoZIzj0DAQcDQgAE5L89jiJKW7bX5P4MxuvO4KN7k1WOJyjKZSrycMZMKWrfOPGNVBBAz3M2wB3bPz0imdjK0ppSyz0GXEWSIglQXw==\",5007168,5533056,10,\"MEUCIQC8wiAAU2/RemIHlxRZ4wkp4QiYpA6yvTFuk2UwBFHd4gIgRjJQqO7ovcVsVYvEFHY0Z+SjTKRCPa2QAyrQiUyZhIs=\"]}]}}}},\"fingerprint\":[\"69:16:99:8B:A7:62:6F:BE:2A:F6:AF:62:E4:DA:4D:8F:32:B8:52:28\"]}", "value_encoding": "ascii", "txid": "fd21e49f5f29de1f38a67201fb009abf8df609fd0b123f7454a04c7f556af7e4", "vout": 0, "address": "NFPRh1m3CPzAoBpbr1YMW2cDqfLit8eheg", "height": 404601, "expires_in": 9896, "expired": false, "ismine": false}, "id": "curltext", "error": null}
   ~~~
1. You can now configure ncdns (or any other application that expects a Namecoin-Core-like `name_show` interface) to connect to Electrum-NMC's RPC server; everything should "just work".
