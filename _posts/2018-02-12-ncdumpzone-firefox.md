---
layout: post
title: "Adding a Firefox TLS Mode to ncdumpzone"
author: Jeremy Rand
tags: [News]
---

Firefox stores its list of certificate overrides in a text file.  While it's not feasible to edit this text file while Firefox is running (Firefox only loads it on startup) I've experimentally found that it is completely feasible to create positive overrides if you shut off Firefox while the override is being created.  But is this a reasonable expectation for Namecoin?  Actually yes.  Here's how we're doing it:

Note that most Namecoin users are doing name lookups via one of the following clients:

* Namecoin Core (in full node mode)
* Namecoin Core (in pruned mode)
* libdohj-namecoin (in `leveldbtxcache` mode)

These have an important feature in common: they all keep track of the UNO (unspent name output) set locally.  That means that you don't need to wait for a DNS lookup to be hooked before you process a TLSA record from the blockchain -- you can process TLSA records in advance, before Firefox even boots!

Note that the above is not true for the following cases:

* libdohj-namecoin (in API server mode)
* `.bit` domains that are delegated to a DNSSEC server

So, what we need is a tool that walks the entire Namecoin UNO set, processes each name, and writes out some data about the TLSA records.  Coincidentally, this is very similar to what `ncdumpzone` does.  `ncdumpzone` is a utility distributed with ncdns.  It exports a DNS zonefile of the `.bit` zone, which is intended for users who for some reason want to use BIND as an authoritative nameserver instead of using ncdns directly.  However, with some minimal tweaking, we can make `ncdumpzone` export the `.bit` zone in some other format... such as a Firefox certificate override list format.

For example, this command:

~~~
./ncdumpzone --format=firefox-override --rpcuser=user --rpcpass=pass > firefox.txt
~~~

Resulted in the following file being saved:

~~~
nf.bit:443	OID.2.16.840.1.101.3.4.2.1	13:E7:03:D6:A2:70:1E:77:41:21:F5:84:6D:3E:0B:FD:5F:00:B7:6B:47:96:82:E3:A2:B0:54:A0:25:76:0A:1A	U	AAAAAAAAAAAAAAAAAAAAAA==
test.veclabs.bit:443	OID.2.16.840.1.101.3.4.2.1	66:86:29:37:ED:53:B3:CE:2B:9B:A5:30:4D:59:83:35:4C:EC:80:9A:1F:39:DC:37:87:6E:00:4B:AF:08:3E:BA	U	AAAAAAAAAAAAAAAAAAAAAA==
www.aoeu2code.bit:443	OID.2.16.840.1.101.3.4.2.1	13:E3:2D:1B:05:B5:DC:57:94:3D:17:EC:99:25:3F:AF:54:87:7E:62:FC:51:18:06:B7:F4:87:51:62:3A:3B:1C	U	AAAAAAAAAAAAAAAAAAAAAA==
markasoftware.bit:443	OID.2.16.840.1.101.3.4.2.1	43:B4:EA:FC:FF:25:CC:85:A9:3D:CE:75:55:31:C9:DB:60:AF:06:C3:65:E5:28:62:08:20:DD:62:F4:70:0E:7D	U	AAAAAAAAAAAAAAAAAAAAAA==
~~~

These 4 lines correspond to the only 4 TLSA records that exist in the Namecoin blockchain right now.  Obviously, the first part of each line is the domain name and port of the website.  `OID.2.16.840.1.101.3.4.2.1` signifies that the fingerprint uses the SHA256 algorithm.  (This is the only one that Firefox has ever supported, but Firefox is designed to be future-proof in case a newer hash function becomes necessary.)  Next comes the fingerprint itself, in uppercase colon-delimited hex.  `U` indicates that the positive override is capable of overriding an "untrusted" error (it can't override validity period or domain name mismatch errors).  The interesting part is `AAAAAAAAAAAAAAAAAAAAAA==`, which Mozilla's source code refers to as a `dbKey`.  Mozilla's source code always calculates this using the issuer and serial number of the certificate.  However, empirically it works just fine if I instead use all 0's (in the same base64 encoding).  Looking at the Mozilla source code, the dbKey isn't actually utilized in the process of checking whether an override exists.  I'm not certain exactly what Mozilla is using it for (it seems to be used in a code path that's related to enumerating all the overrides that exist).  Since the issuer and serial number aren't always derivable from TLSA records (you generally need either a dehydrated certficate or a full certificate for this; my goal here is to work even if only the SHA256 fingerprint of a cert is known), we just set it to all 0's.

Copying the above output into Firefox's cert override file, and then starting up Firefox, I was able to access [the Namecoin forum's `.bit` domain](https://nf.bit/) without any TLS errors.  I've submitted a PR to ncdns.

While I was writing this code, I noticed that ncdns was actually calculating TLSA records incorrectly.  One of the bugs in TLSA record calculation was already known ([Jefferson Carpenter reported it last month](https://github.com/namecoin/ncdns/issues/59)), while the other was unnoticed (the TLSA records contained a rehydrated certificate that accidentally included a FQDN as the domain name; the erroneous trailing period caused the signatures and fingerprints to fail verification).  The fact that these bugs in my TLSA code remained unnoticed for about a year seems to be evidence that no one is actually using TLSA over DNS with Namecoin in the real world; the only Namecoin TLS users are using ncdns's `certinject` feature, which did not have this bug.

It should be noted that this approach isn't secure in the sense that Namecoin TLS with Chromium is, because it doesn't provide negative overrides (meaning that a public CA could issue a malicious `.bit` certificate that wouldn't be blocked by this method).  However, positive and negative overrides are mostly orthogonal goals in terms of implementation, so this is huge progress while we wait for proper WebExtensions support for TLS overrides.  I also think it's likely to be feasible to implement negative overrides using NSS, in a way that Firefox will honor.  More on that in a future post.

This work was funded by NLnet Foundation's Internet Hardening Fund.
