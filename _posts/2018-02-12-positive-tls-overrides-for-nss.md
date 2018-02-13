---
layout: post
title: "Positive TLS Certificate Overrides for NSS"
author: Jeremy Rand
tags: [News]
---

NSS is the TLS implementation used by various applications, including Chromium on GNU/Linux and Firefox on all platforms.  I've finished initial support for positive cert overrides in NSS, and have submitted a PR that is now awaiting review.

I had previously written a WIP PR that implemented positive overrides for NSS, but it worked by using an NSS database directory that was auto-detected based on the active user's home directory.  This seemed like a clever usability trick, but it had 2 severe disadvantages:

1. Some applications don't use the shared NSS database, but instead use their own.  Firefox is one of these applications.
2. For security reasons, we want ncdns to run as its own user with restricted permissions.  This would break the database directory auto-detection.

The new PR has explicit configuration options for which NSS database directory is used.  For example, the following command line config:

~~~
./ncdns -ncdns.namecoinrpcusername=user -ncdns.namecoinrpcpassword=pass -certstore.nss -certstore.nsscertdir="$(pwd)"/certs -certstore.nssdbdir=/home/user/.pki/nssdb -xlog.severity=DEBUG
~~~

Allowed [the Namecoin Forum's `.bit` domain](https://nf.bit/) to load in Chromium in my Fedora VM without any TLS errors.  Obviously, this would need to be combined with the negative override functionality provided by the `tlsrestrict_chromium_tool` program (included with ncdns) in order to actually have reasonable security (otherwise, public TLS CA's could issue `.bit` certs that would still be accepted by Chromium).

Some remaining challenges:

* NSS's `certutil` is extremely slow, due to failure to properly batch operations into sqlite transactions.  I've filed a bug about this with Mozilla.  Until this is fixed, expect an extra ~800ms of latency when accessing Namecoin HTTPS websites.  Possible future workarounds:
    - Run `certutil` sometime before the DNS hook, so that 800ms of latency isn't actually noticeable for the user.  (More on this in a future post.)
    - Do some highly horrifying `LD_PRELOAD` witchcraft in order to fix the crappy sqlite usage.
    - Use a different pkcs11 backend instead of NSS's sqlite3 backend.  (Yes, NSS uses pkcs11 behind the scenes.  More on this in a future post.)
* Firefox doesn't actually respect NSS's trust anchors when the trust anchor is an end-entity certificate.  Possible future workarounds:
    - Use a Firefox-specific positive override mechanism.  (More on this in a future post; also see the WebExtensions posts.)
    - Inject CA certs rather than end-entity certs.  (More on this in a future post.)
* Some NSS applications don't use the sqlite backend, but instead use BerkeleyDB as a backend.  BerkeleyDB can't be opened concurrently by multiple applications, so ncdns can't inject certs while another application is open.  Possible future workarounds:
    - Use an environment variable to force sqlite usage.
    - Run `certutil` while the database isn't open.
    - Use a different pkcs11 backend.
    - Wait for those applications to switch to sqlite.  (Firefox switched in Firefox 58, and it appears more applications may follow suit.)

That said, despite the need for future work, this PR makes Namecoin TLS fully functional in Chromium on GNU/Linux.  (Until negative overrides stop working due to HPKP being removed... more on potential fixes in a future post.)

This work was funded by NLnet Foundation's Internet Hardening Fund.
