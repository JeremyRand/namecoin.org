---
layout: post
title: "More Fun with tlsrestrict_nss_tool on Windows"
author: Jeremy Rand
tags: [News]
---

Last episode: When we last left our hero, tlsrestrict_nss_tool [had a few unfixed bugs]({{site.baseurl}}2018/05/20/testing-tlsrestrict-nss-tool-windows) that made it unusable on Windows.  Everyone believed those bugs would be the final ones.  Were they?  And now, the conclusion to our 2-part special:

Spoiler alert: no, of course they weren't the final bugs!  Obviously Murphy needs to keep showing up, otherwise life as an engineer would be boring, right?

Anyway, so I fixed the 3 known bugs:

1. Use of `cp`
2. No warning when CKBI is empty.
3. Broken Unicode in nicknames.

And at this point, things *almost* worked.  Specifically, I could apply a name constraint that blacklisted `.org`, and accordingly [the Namecoin website](https://www.namecoin.org/) showed an error, while [Technoethical](https://tehnoetic.com/) still worked.  Seems good enough, right?  I certainly thought so, at least enough to announce on #namecoin-dev that I believed it was working.  Except then I had that insane urge to try to torture-test it a bit more.  So I ran tlsrestrict_nss_tool a 2nd time against the same NSS DB and the same CKBI library.  The expected behavior is that it will examine the CKBI and NSS DB, and decide that no additional cross-signing is needed.  Unfortunately, I instead was treated to a fatal error due to an ASN.1 parse error, specifically due to trailing data.

I've seen this error before, and it's usually triggered by an NSS quirk.  NSS doesn't actually keep track of each certificate uniquely.  If you put 2 certificates in an NSS DB that have the same Subject, and you ask certutil to give you one of them (doesn't matter which), certutil will actually give you *both of them*, concatenated.  This happens regularly in our usage, because the cross-signed CA and the (distrusted) original CA have the same Subject (by design).

Further examination of the logs showed that the errors were showing up while trying to handle certs that had a very odd characteristic: their names looked like what you would get from concatenating the Namecoin prefix with an empty string instead of with the name of the certificate.  Given that I had just spent time fixing issues with Unicode encoding of certificate names, this seemed to be a likely culprit.

So, I made 2 (overdue anyway) changes to the codebase:

1. Switch from DER to PEM encoding when communicating with certutil.  DER doesn't have boundaries when you concatenate certificates, while PEM does.  Using PEM should make debugging a lot easier when multiple certs show up with the same name.
2. When dumping a PEM cert from the NSS DB, explicitly check for multiple PEM certs, and if more than one is present, try to guess which one is correct by checking for the Namecoin prefix in its Subject CommonName and Issuer CommonName (this will be unambiguous under typical conditions).  If more than one cert is present that matches the expected prefixes, throw an error and log all of the PEM certs that showed up in the dump.

So, with those changes added, I ran it again, and quickly got an error telling me that 9 certs were being returned in a single dump.  How odd.  Conveniently, the log told me what certificate it was trying to dump when this happened: "Namecoin Restricted CKBI Intermediate CA for ePKI Root Certification Authority".  This didn't look like a Unicode issue at all -- that name is entirely Latin.  So I Googled for "ePKI Root Certification Authority", and quickly facepalmed.  That root CA *doesn't have a CommonName!*  Suddenly the symptoms made sense.  The root and intermediate CA's that are created by cross_sign_name_constraint_tool prepend a Namecoin string to the CommonName of the input CA and discard the rest of the input CA's Subject, meaning that if multiple input CA's have a blank CommonName, their resulting Namecoin root and intermediate CA's will end up with colliding Subjects.  Fail.

The fix, of course, is to append the SHA256 fingerprint of the input CA to the Subject CommonName of the root and intermediate Namecoin CA's.  This ensures that we'll get a unique Subject per input certificate.

And now, it works.  Repeated runs of tlsrestrict_nss_tool work as they should.  Kind of irritating to spend so much time chasing a silly fail like that, but on the bright side the switch to PEM resulted in cleaner code.

Next, I'll be integrating tlsrestrict_nss_tool into ncdns.  Hopefully this will expose any remaining weirdness.

This work was funded by NLnet Foundation's Internet Hardening Fund.
