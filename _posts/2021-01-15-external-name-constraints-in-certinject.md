---
layout: post
title: "External Name Constraints in certinject"
author: Jeremy Rand
tags: [News]
---

In my [previous post]({{ "/2021/01/14/undocumented-windows-feature-external-name-constraints.html" | relative_url }}), I introduced the undocumented Windows feature for external name constraints, which allow us to apply a name constraint without the consent of a CA, and without needing to cross-sign the CA.  I mentioned that the Windows utility `certutil` can be tricked into setting this Property on a certificate using the `-repairstore` command.  Alas, abusing `certutil` to do this comes with some problems:

1. The `-repairstore` command looks suspiciously like it's intended to be used for totally different purposes, and I simply do not trust it to only set a name constraint and nothing else.
2. Using `certutil` to edit a public CA requires Administrator privileges, which is not great from a sandboxing perspective.
3. We'd ideally like to apply the name constraint to *all* of the root CA's in the certificate store, not just a single one.  The `certutil` API doesn't exactly make this *impossible*, but the UX is a lot worse than, say, the `tlsrestrictnss` tool I wrote that does something comparable for the NSS TLS library.

Those of you who've looked at the ncdns source code will recall that ncdns includes a library called `certinject`, which is designed to interact with Windows cert stores without needing Administrator privileges.  Since `certutil` isn't free software, but `certinject` is, and `certinject` already solves the privilege issue for us, Aerth and I have been extending `certinject` to do what's needed for external name constraints.

## Injecting to Arbitrary Cert Stores

The original `certinject` code only could write to the `Root` logical cert store in the `Enterprise` physical cert store.  This is where root CA's added by the system administrator would, by convention, be stored.  This is fine for `certinject`'s original purpose of injecting self-signed certs for Namecoin websites, but all the CA's we want to apply a name constraint to are elsewhere: generally in either the `Root` or `AuthRoot` logical cert store in the `System` physical cert store.  (`Root` is where the Microsoft root CA's live; `AuthRoot` is where the root CA's run by non-Microsoft corporations live.)  `certinject` can now inject certs into arbitrary logical stores (including the `Disallowed` logical store, which is used to mark a certificate as revoked), and now supports the `System` and `Group-Policy` physical stores in addition to the previously supported `Enterprise` physical store.

## Serializing Properties

Originally, `certinject` only serialized Blobs that included the DER-encoded certificate, with no other Properties set.  This worked fine for its original purpose, but we needed to make it more flexible.  `certinject` can now serialize Blobs with arbitrary Properties set.  `certinject`'s list of Properties supported by Windows is generated (to Go source code) via a Bash script (lots of `grep` and `sed`) using the `wincrypt.h` file from ReactOS as input.  Curiously, `wincrypt.h` only seems to contain the Properties supported by Windows XP.  A bunch of extra Properties were added in Windows Vista, but these are only listed in another header file, `certenroll.h`, which does not appear to exist in the ReactOS source code yet.  At the moment, we don't have any urgent need for the Vista Properties, so we're sticking with the ReactOS header file for maximum free-software-ness.

`certinject` can also now generate the binary data for Properties involving either EKU (extended key usage) or name constraints.  The Golang standard library doesn't exactly make this easy; we settled on the approach of setting the EKU or name constraint fields in an `x509.Certificate` template, serializing the entire certificate, deserializing the result back to a template, and searching the template's list of parsed `Extensions` for something that matched the OID of either EKU or name constraints.  A tad inefficient, but this approach does seem to maximize the usage of stable, production-grade standard library API's compared to custom parsing code on our end, so I think it's safer than trying to roll our own super-efficient implementation.

While I was writing the EKU serialization code, I noticed that the Go standard library supports two EKU values that I had never heard of: `ExtKeyUsageMicrosoftServerGatedCrypto` and `ExtKeyUsageNetscapeServerGatedCrypto`.  Some DDG-ing revealed that these are a historical relic of 1990's-era export-grade cryptography (also referred to as "International Step-Up").  In particular, I found a Mozilla Bugzilla bug indicating that modern TLS implementations still support these 1990's-era abominations because there exist public CA certs that rely on those particular EKU values, which didn't expire until 2020.  I decided to explicitly not support these EKU values in `certinject`; users who desire to see some mildly more colorful language describing my opinion of this functionality can grep the `certinject` source code for "beehive".

We intend to add support for more Properties later; there are several other Properties that have caught our eye, although they're not a very high priority for us right now.

The only other notable issue we encountered in Blob serialization is that we found experimentally that the Property containing the DER-encoded certificate **must** be the final Property in the serialized Blob.  Any Properties that come after the DER-encoded certificate will be silently ignored.  (This meant that our first try, which sorted Properties by ascending Property ID, didn't work as intended, because the name constraint Property has a greater Property ID than the DER-encoded certificate.  Oops!)  Interestingly, Aerth was able to dig up a Microsoft documentation page indicating that listing any Properties after the certificate itself would yield undefined behavior.  I guess now we know what that undefined behavior is.

## Editing Existing Blobs

`certinject` was always a one-way thing: you put in the cert you want, and it spits out a Blob.  However, this is not really desirable for the purpose of applying an external name constraint, because we don't want to overwrite the Properties that already exist in the certificate store.  (Many of them, e.g. EKU, are probably security-critical.)  So we added functionality to read an existing Blob from the cert store and use that as the starting point, instead of creating an empty Blob with just the DER-encoded certificate.

This is not as complete as it could be.  In the future, we intend to support fine-grained editing within a Property, e.g. adding a domain name to the name constraints list without destroying the existing name constraints.

## Searching for Certificates by Hash

Originally, `certinject` was designed to accept a certificate as input, and it would then calculate the SHA1 hash itself for use as a Windows registry key.  (Yes, Windows still uses SHA1 as a certificate identifier.  Yes, this is stupid.  No, it's not our problem.)  This made sense when the intent was to inject a cert that previously didn't exist in the cert store, but since we now want to use it to edit existing certs, there's no reason to force the user to know the preimage.  `certinject` now allows the user to specify a SHA1 hash of a certificate, which will get passed through directly when constructing the registry key.  This makes things a lot easier, and also leads to the next item.

## Applying Operations Store-Wide

Our intent for external name constraints is to Constrain All The Things, i.e. we want **all** built-in root CA's to be prohibited from issuing certificates for `.bit` domains.  Put another way, for those of you who've worked with our NSS cross-signing tools, we want the UX to be more like `tlsrestrictnss` and not `crosssignnameconstraint`.  `certinject` can now crawl the list of all certificates in a given certificate store, and apply the specified EKU or name constraint operations to all of them.  For practical purposes, this would usually entail applying the name constraint to all certs in the `Root` and `AuthRoot` logical stores of the `System` physical store.  Optionally, users might also want to apply the name constraint to the `Root` logical store of the `Enterprise` and `Group-Policy` physical stores.

This is not quite as complete as it could be.  In the future, we intend to support tagging certificates with a "magic value", which allows users to designate specific root CA's that they want to exempt from the name constraint; `certinject` would then avoid applying the name constraint to those specific CA's when doing a store-wide operation.  This could be used to allow users to deliberately run a MITM proxy on `.bit` traffic for diagnostic purposes, without losing protection from other root CA's that the user doesn't want to intercept `.bit` traffic.  (`certinject` already has some skeleton support for magic values, which ncdns uses for cleaning up expired certificates for privacy purposes.  So this will not be hard to add.)

## Adding a Command-line Utility

`certinject` now can be built as a command-line `certinject.exe` utility, which facilitates users who want to use it as an alternative to `certutil` from the command-line.  It also supports both DER and PEM certficates, for maximal ease of use.

## Adding Extensive Integration Tests

Cirrus CI now tests `certinject` daily for a variety of use cases against a real TLS client in a real Windows VM.  This has already surfaced a few bugs (which we fixed), and also ensures that if Microsoft ships an update that changes the behavior we're relying on, we will automatically get notified.

An interesting issue that we encountered here was that Windows tends to cache TLS cert verification results, which caused integration tests to interfere with each other.  This caching is generally done on a per-process basis, so spawning a separate PowerShell process for each TLS handshake worked pretty well for avoiding this issue.  (An exception is that Windows has some kind of special caching mechanism for "revoked" status, which is not on a per-process basis.  This meant that our tests for the `Disallowed` logical store needed to take extra precautions, specifically sleeping for 30 seconds between consecutive TLS handshakes, so that the cache would expire.  Yes, this is stupid.)

Another more boring (yet also more amusing) issue we found is that our initial on-a-whim choice of using `github.com` as a test case of a TLS cert that was publicly trusted backfired, because `github.com` promptly blacklisted our CI VM for rate-limit violations, causing our tests to start failing nondeterministically as soon as we had more than a few test cases.  Oops.  We switched these tests to use `namecoin.org`, which seems a tad more ethical, and all is okay now.

We also did a bunch of code cleanup, based on feedback from static analyzers.

## So what's next?

Stay tuned for a future post on how we're going to integrate this new `certinject` functionality into Namecoin.
