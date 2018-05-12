---
layout: post
title: "Fixing DNAME records in madns and dns-prop279"
author: Jeremy Rand
tags: [News]
---

One of the more obscure DNS record types is `DNAME` (AKA the Namecoin `"translate"` JSON field), which is basically a DNS redirect for an entire subtree.  For example, currently `radio.bit.` has a `DNAME` record pointing to `biteater.dtdns.net.`, which means that any subdomain (e.g. `batman.radio.bit.`) becomes a `CNAME` redirect (e.g. to `batman.biteater.dtdns.net.`).

`DNAME` is not exactly a favorite of mine in the context of Namecoin, because it's easy to misuse it in a way that assigns trust for a Namecoin domain name to 3rd party keys whom Namecoin is intended to not trust (e.g. if you `DNAME` `radio.bit.` to a DNS domain name, you're also assigning control of the `TLSA` records for `_443._tcp.radio.bit.` to whatever DNSSEC keys have the ability to sign for that DNS domain name, which probably includes a DNS registrar, a DNS registry, and the ICANN root key).  That said, `DNAME` is part of the DNS, and so it *should* work in Namecoin, even though there aren't likely to be many good uses for it in Namecoin.

Which is why I was surprised to notice when I tested `DNAME` today that it wasn't actually working as intended in ncdns or dns-prop279.  Some digging revealed that madns (the authoritative DNS server library that ncdns utilizes) didn't actually have any `DNAME` support; the place in the code where it should have gone was just marked "TODO".  This was a great excuse for me to get my feet wet with the madns codebase (Hugo usually handles that code), so I jumped in.

In the process of adding `DNAME` support to madns, I got to read [RFC 6672](https://tools.ietf.org/html/rfc6672#section-2.3), and noticed that it very much looks like Namecoin's `d/` (domain names JSON) spec is not quite compliant with the RFC.  Specifically, the [Namecoin spec](https://github.com/namecoin/proposals/blob/1b0043a98fe8f4cf1a85ec92fdbe98d38b5886b3/ifa-0001.md#item-suppression-rules) says that a `DNAME` at `radio.bit.` suppresses all other records at `radio.bit.`, whereas the RFC says that other record types can coexist at `radio.bit.`, with the sole exception of `CNAME` records.  I've filed a bug to get the Namecoin spec brought in line with the RFC.

Once I got madns supporting `DNAME` properly, that meant I could test dns-prop279 with `DNAME`.  Except testing quickly showed that dns-prop279 was crashing when it encountered a `DNAME`.  A quick check of the stack trace showed that I had made a minor screw-up in the error checking in dns-prop279 (specifically, dns-prop279 is asking for a `CNAME`, but doesn't properly handle the case where it receives both a `DNAME` and a `CNAME`).  A quick bugfix later, and dns-prop279 was correctly handling `DNAME`.

The fixes are expected to be included in the next release of ncdns and dns-prop279.

This work was funded by NLnet Foundation's Internet Hardening Fund.

(Side note: some readers might have noticed that I was posting less frequently over the past month or so.  That's because my master's thesis defense was on May 3, and as a result I spent most of the last month getting ready for that.  I passed my defense, so things should be back to normal soon.)
