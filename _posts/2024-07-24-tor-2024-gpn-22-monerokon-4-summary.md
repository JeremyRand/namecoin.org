---
layout: post
title: "Tor Meeting 2024 / GPN 22 / MoneroKon 4 Summary (UPDATED)"
author: Jeremy Rand
tags: [News]
---

As was [previously announced]({{ "/2024/05/15/tor-2024-gpn-22-monerokon-4.html" | relative_url }}), Rose, yanmaani, Hugo, and I (Jeremy Rand) represented Namecoin at three conferences in Europe. This was an excellent series of events for a number of reasons, not least the fact that this is the first time we've ever had 4 Namecoin developers attending a single conference (our previous record was 3 developers, at 37C3). Expanding our team size has been a priority for a while, and it's great to see that this is starting to pay off. Of course, there were major substantive benefits as well. As usual for conferences that Namecoiners attend, we engaged in a large number of conversations with other attendees.  Also as usual, we won't be publicly disclosing the content of those conversations, because I want people to be able to talk to us at conferences without worrying that off-the-cuff comments will be broadcast to the public.

Notes/slides for Namecoin's sessions at the Tor meeting, and recordings of Namecoin's sessions at GPN and MoneroKon, are published below:

## Tor Lisbon 2024: Self-authenticating TLS Certificates for Onion Services using a PKCS#11 module

Facilitator: Jeremy Rand

Onion services are self-authenticating. It is highly unfortunate that this isn't presently the case for their TLS certificates. ahf already made a proof of concept for using onion service Ed25519 keys as a TLS CA, but this requires substantial browser modifications, both to the trust logic (browsers don't know they should trust these keys) and to the crypto support (browsers don't support EdDSA). However, there is hope! Mainstream browsers (including Firefox and Chromium) actually do expose APIs that (with a little creativity) can enable validating TLS certificates based on onion service keys. You don't even need to recompile the browser. Let's discuss the implications of this, and maybe make some plans!

[Notes and slides are here.](https://gitlab.torproject.org/tpo/team/-/wikis/Meetings/2024/Lisbon/self-auth-certs-for-onion-services)

## Tor Lisbon 2024: Pluggable Transport Executables Compression

Facilitators: Jeremy Rand, Morgan Ava

The download size of Tor Browser is somewhat larger than would be ideal (even causing rejection in the Android Play Store recently). The Go-based pluggable transports take up a nontrivial portion. A potential solution lies within the well-regarded u-root project (a spin-off of Coreboot that one of Tor's interns worked with recently). u-root includes a subproject called Go-Busybox, which can merge multiple Go executables into one meta-executable, thus de-duplicating library code via the standard compiler optimization process. Our initial experiments have shown significant space savings, and the savings further compound if Namecoin naming (which also uses Go) is included in the Tor Browser binaries. Is this a fruitful line of inquiry? Are there security implications that we should consider? Are there other tricks we can use to decrease the size of the PT binaries? Let's discuss!

[Notes and slides are here.](https://gitlab.torproject.org/tpo/team/-/wikis/Meetings/2024/Lisbon/pluggable-transport-executables-compression)

## Tor Lisbon 2024: Refreshing Best Practices Guidelines for Tor-Friendly Applications

Facilitators: Jeremy Rand, Morgan Ava

Many years ago, some community members wrote up a Tor-Friendly Applications Best Practices document. The document has proven to be valuable, but parts are underspecified, and other parts will need revamping for Arti. This session is to refresh the document. Bring your ideas for things that Tor-friendly applications should do, and your horror stories of things Tor-friendly applications shouldn't do.

[Notes are here.](https://gitlab.torproject.org/tpo/team/-/wikis/Meetings/2024/Lisbon/refreshing-best-practices-guidelines-for-tor-friendly-applications)

## GPN 22: Self-Authenticating TLS Certificates for Tor Onion Services

Speaker: Jeremy Rand

TLS (the security layer behind HTTPS) and Tor onion services (anonymously hosted TCP services) are both excellent protocols. Wouldn't it be nice if we could use them together? In this talk, I'll cover a working implementation of combining TLS with onion services, without compromising on the security properties that each provides.

Topics to be covered include:

* Why would you want to combine TLS with onion services? Why isn't onion service encryption good enough?
* Why isn't unauthenticated TLS (e.g. self-signed certificates) good enough for onion services?
* How can we authenticate a TLS certificate for a .onion domain without relying on public CA's like Let's Encrypt or any other trusted third parties? (No we're not using a blockchain.)
* How can we teach standard (unmodified) web browsers like Firefox to apply different certificate validation logic for .onion certificates?
* How can we teach standard (unmodified) web browsers like Firefox to validate certificates using typically-unsupported elliptic curves like Ed25519 (which Tor uses)?

[Video is here.](https://media.ccc.de/v/gpn22-469-self-authenticating-tls-certificates-for-tor-onion-services)

## MoneroKon 4: Human-Meaningful, Trustless, Anonymous Monero Addresses with Namecoin

Speaker: Jeremy Rand

Monero addresses are already long and unwieldy, and they’re about to get longer with Jamtis. Namecoin is a DNS-like naming system implemented as the first project forked from Bitcoin, predating Monero by 3 years to the day. This talk will cover using Namecoin as a human-meaningful naming layer for Monero addresses, recent anonymity advances that make Namecoin’s privacy more suitable for this use case, how OpenAlias fits in, and how Namecoin compares to the MoneroDNS approach of creating a Monero sidechain for this purpose.

[Video is here.](https://www.youtube.com/watch?v=qQ4ptuh_w84)

## Thanks

Huge thank you to the following groups who facilitated our participation:

* Cyphrs
* [The Replicant Assembly](https://replicant.us/)
* Cypherpunks

We're hoping to return to these events next year.

This work was funded by NLnet Foundation's Internet Hardening Fund and Cyphrs.
