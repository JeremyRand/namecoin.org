---
layout: post
title: "Namecoin Receives 30k EUR in Additional Funding from NLnet Foundation’s Internet Hardening Fund and Netherlands Ministry of Economic Affairs and Climate Policy"
author: "Jeremy Rand"
tags: [News]
---

We're happy to announce that Namecoin is receiving 30,000 EUR (roughly 33,306 USD) in additional funding from [NLnet Foundation](https://nlnet.nl/)'s Internet Hardening Fund.  If you're unfamiliar with NLnet, you might want to read [about NLnet Foundation](https://nlnet.nl/foundation/), or just take a look at [the projects they've funded over the years](https://nlnet.nl/thema/index.html) (you might see some familiar names).  The [Internet Hardening Fund](https://nlnet.nl/internethardening/) is managed by NLnet and funded by the [Netherlands Ministry of Economic Affairs and Climate Policy](https://www.rijksoverheid.nl/ministeries/ministerie-van-economische-zaken-en-klimaat).  Unlike our [already-active funding]({{ "/2021/04/30/funding-from-nlnet.html" | relative_url }}) from NLnet's NGI0 Discovery Fund, which is focused on Namecoin Core and Electrum-NMC, this new funding is focused on TLS use cases.

We will use this funding to work on the following:

* Namecoin interoperability with TLS implementations that use AIA.  In addition to reducing attack surface of Namecoin TLS on Windows, this lays the groundwork for Namecoin TLS support on Android/Linux, macOS, and iOS.
* Refactoring Namecoin TLS PKCS#11 interoperability to use higher-level API's that are more straightforward to audit.  In addition to improving security and reliability of Namecoin TLS on Firefox (all OS's) and GNU/Linux (most browsers), this lays the groundwork for things like decentralized TLS with `.onion` domains (in collaboration with the Tor developers), and is likely to benefit 3rd-party developers who wish to create PKCS#11 modules in Go (e.g. for HSM or smart card use cases).
* Adding imposed name constraint support to NSS's TLS trust store (used by Firefox), similar to the existing functionality that exists in CryptoAPI (used by Windows) and GnuTLS (used by GNOME Web).  In addition to allowing us to simplify Namecoin's TLS interoperability code considerably, this is likely to benefit system-wide trust policy projects such as sponsored by Red Hat.
* Auditing TLS clients for name constraint bugs, using Netflix's BetterTLS test suite.  This helps ensure that Namecoin's security guarantees cannot be bypassed by exploiting bugs in 3rd-party applications.
* Improved integration testing for Namecoin TLS PKCS#11 interoperability.  In addition to helping us review PR's faster, this helps ensure that future changes in third-party applications/libraries don't cause problems for Namecoin.
* Windows installer for Namecoin TLS in Firefox and/or Tor Browser.

This work will principally be done by developers Aerth and Jeremy Rand (who is the author of this post).

We'd like to thank the awesome people at NLnet Foundation for the continued vote of confidence, as well as the Netherlands government for recognizing that the economy and the climate require a hardened Internet.

We'll be posting updates regularly as development proceeds.  Due to delays in paperwork processing, some work covered by this funding has already taken place; for transparency, we have amended the relevant past news posts to clarify that they were funded by Internet Hardening Fund.
