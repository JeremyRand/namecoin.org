---
layout: post
title: "cross_sign_name_constraint_tool Drops Support for Go v1.9.x; Users Who Self-Built It With Go v1.9.x Should Update Immediately"
author: Jeremy Rand
tags: [News]
---

`cross_sign_name_constraint_tool`, as you may remember, is a Namecoin-developed tool that applies name constraints to a certificate authority, without requiring any permission from that CA.  It can be used to prevent malicious CA's from issuing certificates for Namecoin domain names, even if those CA's are trusted for DNS domain names.  `cross_sign_name_constraint_tool` (or its underlying library) is used by other Namecoin projects, such as `tlsrestrict_nss_tool` (which integrates `cross_sign_name_constraint_tool` with the NSS cert store used by Firefox and the GNU/Linux version of Chromium) and ncdns (which will soon make `tlsrestrict_nss_tool` easy to use on Windows).  `cross_sign_name_constraint_tool`'s official binaries are produced using Go v1.10.x.  Recently, I happened to notice a bug in Go v1.9.x (which is fixed in Go v1.10.0 and higher) that causes `cross_sign_name_constraint_tool` to fail with an error like this:

~~~
Error generating cross-signed CA: Couldn't unmarshal original certificate: asn1: structure error: tags don't match (2 vs {class:0 tag:16 length:13 isCompound:true}) {optional:false explicit:false application:false defaultValue:<nil> tag:<nil> stringType:0 timeType:0 set:false omitEmpty:false}  @21
~~~

The error only occurs with a small subset of CA certificates; the specific CA that triggered this error for me was `Verisign Class 3 Public Primary Certification Authority - G3` (which is trusted by Fedora).

This bug doesn't affect users of our official binaries, because the official binaries weren't built with an affected Go version.  My WIP ncdns PR for automatically integrating `tlsrestrict_nss_tool` would have made such an error very obvious: it causes ncdns to stop resolving `.bit` domains when such an error is observed, so there's no risk to users from name constraints being incompletely applied.  However, users who built their own `cross_sign_name_constraint_tool` or `tlsrestrict_nss_tool` using Go 1.9.x, and who failed to notice a prominent error message when running it, would potentially be left incompletely protected from the threats that `cross_sign_name_constraint_tool` is designed to protect against.

If you're running a self-built `cross_sign_name_constraint_tool` or `tlsrestrict_nss_tool` that was built with Go 1.9.x, I strongly recommend that you rebuild with Go 1.10.x and re-apply your desired name constraints.  I don't have any reason to believe that any CA has exploited this in the wild.  (Any CA who tried to exploit it would probably have been detected by certificate transparency.)  And remember, `cross_sign_name_constraint_tool` is designed to protect against attacks that users of the DNS are *completely* vulnerable to -- this bug is not a downgrade from the security model of the DNS.  (Indeed, even with this bug, affected users were still protected from a random subset of their system's trusted CA's, so security in this aspect was still somewhat higher than that of the DNS.)

I've submitted a test case to `cross_sign_name_constraint_tool`, which will ensure that anyone who tries to build it with an affected Go version will receive a test failure.  ncdns will also be updated to use Go 1.10.x prior to the release that integrates `tlsrestrict_nss_tool`.

This work was funded by NLnet Foundation's Internet Hardening Fund.
