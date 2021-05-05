---
layout: post
title: "Deterministic salts in Namecoin Core"
author: yanmaani
tags: [News]
---

Namecoin Core will, starting version 0.22, no longer require that a salt or TXID be provided in the `name_firstupdate` RPC call. If no transaction ID is provided, the wallet will perform a linear scan over its unspent outputs to attempt to find a matching transaction. If no salt is provided, it will assume that it can be deterministically generated from the private key using the same scheme already implemented in [Electrum-NMC](/2020/05/10/deterministic-salts.html).

To facilitate this, `name_new` has also been changed to use deterministically generated salts.

The old API is still supported; users are free to manually enter a salt, TXID, both, or neither. In fact, for `name_new` transactions created by versions of Namecoin Core prior to this change, the salt is still required. However, the transaction ID can always be determined automatically. In principle, it should only be necessary to provide the transaction ID to select which of several possible `name_new` outputs should be spent.

There is therefore no longer any need to write down the salt and transaction ID going forward. With the new API, registering a name from the RPC console works like this:

```
name_new "d/myname"
(wait 12 blocks...)
name_firstupdate "d/myname" "value"
```

This change is being made as part of the effort to simplify the RPC API for name management. It was also one of the prerequisites for a new name registration GUI, work on which is intended to proceed shortly.

This work was funded by NLnet Foundation's NGI0 Discovery Fund, which is funded by the European Commission, the executive branch of the European Union.
