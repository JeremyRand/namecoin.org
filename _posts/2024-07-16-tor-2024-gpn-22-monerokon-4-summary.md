---
layout: post
title: "Tor Meeting 2024 / GPN 22 / MoneroKon 4 Summary"
author: Jeremy Rand
tags: [News]
---

As was [previously announced]({{ "/2024/05/15/tor-2024-gpn-22-monerokon-4.html" | relative_url }}), Rose, yanmaani, Hugo, and I (Jeremy Rand) represented Namecoin at three conferences in Europe. This was an excellent series of events for a number of reasons, not least the fact that this is the first time we've ever had 4 Namecoin developers attending a single conference (our previous record was 3 developers, at 37C3). Expanding our team size has been a priority for a while, and it's great to see that this is starting to pay off. Of course, there were major substantive benefits as well. As usual for conferences that Namecoiners attend, we engaged in a large number of conversations with other attendees.  Also as usual, we won't be publicly disclosing the content of those conversations, because I want people to be able to talk to us at conferences without worrying that off-the-cuff comments will be broadcast to the public.

Sessions at the Tor meetings are not recorded. However, my presentations at GPN and MoneroKon were recorded, and are published below:

## GPN 22: Self-Authenticating TLS Certificates for Tor Onion Services

Speaker: Jeremy Rand

TLS (the security layer behind HTTPS) and Tor onion services (anonymously hosted TCP services) are both excellent protocols. Wouldn't it be nice if we could use them together? In this talk, I'll cover a working implementation of combining TLS with onion services, without compromising on the security properties that each provides.

Topics to be covered include:

* Why would you want to combine TLS with onion services? Why isn't onion service encryption good enough?
* Why isn't unauthenticated TLS (e.g. self-signed certificates) good enough for onion services?
* How can we authenticate a TLS certificate for a .onion domain without relying on public CA's like Let's Encrypt or any other trusted third parties? (No we're not using a blockchain.)
* How can we teach standard (unmodified) web browsers like Firefox to apply different certificate validation logic for .onion certificates?
* How can we teach standard (unmodified) web browsers like Firefox to validate certificates using typically-unsupported elliptic curves like Ed25519 (which Tor uses)?*

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
