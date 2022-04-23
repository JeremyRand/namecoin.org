---
layout: post
title: "CTLPop: Populating the Windows AuthRoot Certificate Store"
author: Jeremy Rand
tags: [News]
---

In my [previous post]({{ "/2021/01/15/external-name-constraints-in-certinject.html" | relative_url }}), I introduced improvements to `certinject`, which allow us to apply a name constraint to all certificates in a Windows certificate store, without needing Administrator privileges.  Alas, there is a major issue with using `certinject` as presented in that post.  The issue is that most of the built-in root CA's in Windows *aren't part of any cert store!*

## Wait, What?

Yep.  Allow me to explain.  The root CA list in Windows is pretty large (420 root CA's at the moment, if my count is correct), and can also be updated on the fly via Windows Update.  However, due to what Microsoft claims are performance concerns, only 24 of those root CA's are actually populated in the cert store on a fresh Windows installation.  The full list lives in a file called `AuthRoot.stl`, which ships with Windows, and can also be updated via Windows Update.  This file is a *CTL* (certificate trust list), meaning it only stores the hashes of the certificates (and a bit of other metadata), not the certificate preimages.  (Why does a "CTL" have the file extension `.stl` rather than `.ctl`?  Don't ask.  Just accept that Microsoft hates you.  It'll be easier that way.)  When Windows tries to verify a certificate that chains back to a certificate in `AuthRoot.stl` but isn't in the Windows certificate store, it automatically fetches the certificate preimage from Windows Update, and inserts it into the certificate store (typically in the `AuthRoot` logical store) prior to proceeding with the verification.  This is all transparent to the user under typical circumstances.

Personally I am highly dubious that this is a meaningful performance optimization, especially since this system was created (AFAICT) around 2 decades ago, so even if it helped performance when it was introduced, I doubt that this performance gain is applicable on modern hardware (especially since today, network latency is a much bigger contributor to perceived performance than any kind of local CPU or IO performance, and this system entails extra network latency when verifying certificates with a previously-unseen root CA).

## So What's the Problem?

Well, unfortunately, if a root CA is being downloaded on-the-fly during certificate verification, that prevents `certinject` from applying a name constraint to it before it gets used.  This is unfortunate, since it means that most of the root CA's in Windows cannot be reliably constrained via `certinject`.

## What Can We Do?

It's entirely feasible to download the full list of certificate preimages that `AuthRoot.stl` refers to.  There's even a mostly-undocumented [1] `certutil` [command](https://docs.microsoft.com/en-us/previous-versions/windows/it-pro/windows-server-2012-R2-and-2012/dn265983(v=ws.11)#new-certutil-options) for this.  But what do we want to do with it?  Two ideas occurred to me:

1. We could download the full set of certs in SST (serialized store) format (which also includes all the metadata, i.e. Properties besides the cert preimage), and ask `certutil` to import it to the `AuthRoot` logical store.  Unfortunately, this means we'd need to run as Administrator, which is not really ideal.  Also, how do we know for sure that `AuthRoot` is the right logical store to import them?  Seems suboptimal.
2. We could download the full set of certs to individual files, and use `certinject` to individually inject them to the `AuthRoot` store, with Properties manually parsed from the `AuthRoot.stl` file.  While this does avoid running with Administrator privileges by virtue of using `certinject`, it means we'd have to carefully parse the Properties from `AuthRoot.stl`, and make sure that `certinject` is applying them correctly.  Seems like a lot of attack surface.  Also, we still don't know for sure that they all belong in the `AuthRoot` logical store, and it still requires write privileges to the `AuthRoot` store's registry key.  So this is still not great.

Since neither of these jumped out at me as "obviously this is the right way to do it", I came up with another idea.

## Make Windows Do Our Job For Us!

If you were reading the first section of this post carefully, you might have noted that Windows itself will happily insert the certificates from `AuthRoot.stl` into the certificate store under certain circumstances, specifically when it's necessary for certificate verification.  Hmm, this sounds like something we can abuse!  What happens if we download the full set of certificates to individual files from Windows Update, and then just politely ask `certutil` to **verify** all of them?  Intuitively, this seems like the kind of thing that will cause the following Series of ~~un~~Fortunate Events:

1. CryptoAPI tries to verify the certificate by chaining it to a root CA in the certificate store.  This, naturally, fails.
2. CryptoAPI checks whether the certificate chains to a root CA in `AuthRoot.stl`.  Yes, the certificate does claim to be issued by a root CA in `AuthRoot.stl`.  (In reality, that's because the certificate is issued by **itself**, but CryptoAPI can't know this until it sees the issuing certificate.)
3. CryptoAPI helpfully fetches the root CA referenced by `AuthRoot.stl` from Windows Update, and adds it to the certificate store.  **Yay, we've achieved our goal!**
4. CryptoAPI discovers that the certificate we're trying to verify is now marked as trusted, and `certutil` tells us that verification succeeded.  That's cool and such, but we don't really care about this step.

And the beauty of this trick is that we don't need **any** elevated permissions for the certificate store (all we did was ask Windows to verify some certificates, which is obviously an unprivileged operation; Windows messed with the certificate store for us), nor did we need to worry about the Property metadata (again, Windows does this for us; there's nothing we can screw up there no matter how buggy our code is).

And indeed, based on testing, the above workflow works exactly as I was hoping it would!  Running `certutil` to verify a certificate in `AuthRoot.stl` downloaded from Windows Update does indeed result in the certificate being immediately imported to the certificate store.  How cool is that?

(Side note: it turns out I was absolutely right to be wary of assuming that the `AuthRoot` logical store is the right place.  In fact, the `AuthRoot.stl` CTL also covers a small number of Microsoft-operated root CA's, which go in the `Root` logical store -- `AuthRoot` is only for built-in root CA's **not** operated by Microsoft.)

## CTLPop: the AuthRoot Certificate Trust List Populator

I've created a simple PowerShell script called CTLPop, which automates this procedure.  Just create a temporary directory (e.g. `.\place-to-store-certs`) to download certificates to, run `ctlpop.ps1 -sync_dir .\place-to-store-certs`, wait a few minutes (Travis CI indicates that it takes 4 minutes and 26 seconds to run twice in a row on their VM), and voila: now all 420 of the built-in root CA's are part of your certificate store, ready for you to apply name constraints via `certinject`!

## So What's Next?

The easiest way to use CTLPop and `certinject` is to simply run CTLPop once as part of the ncdns NSIS installer, and then run `certinject` to apply the name constraint globally (again, as part of the NSIS installer).  This is probably what we'll ship initially, since it's very simple and mostly works fine.  However, it's not great in terms of sandboxing (the NSIS installer runs as Administrator), and it's also not as robust as I'd like (because if Microsoft updates the `AuthRoot.stl` list later, the new root CA's won't get the name constraint unless ncdns is reinstalled).  The "right" way to do this is to have a daemon that continuously watches for `AuthRoot.stl` updates,  and runs CTLPop and `certinject` whenever an update is observed.  We could even add a dead-man's switch to make ncdns automatically stop resolving `.bit` domains if the `AuthRoot.stl`-watching daemon encounters some kind of unexpected error.  We'll probably migrate to this approach in the future, since it's much more friendly to sandboxing (CTLPop, which involves network access and parsing untrusted data via `certutil`, can run completely unprivileged, and `certinject`, which does not touch the network and doesn't parse untrusted data, only needs read+write privileges to the `Root` and `AuthRoot` certificate stores), and also will handle new root CA's gracefully.

Expect to see name constraints via CTLPop and `certinject` coming soon to an ncdns for Windows installer near you!

## Epilogue

After this post was written, but before publication, I discovered that `certutil` actually has a built-in command that will do exactly the same thing as CTLPop: `certutil -v -f -verifyCTL AuthRootWU`.  So, we can scrap the custom PowerShell implementation I wrote.  Everything else, e.g. integration with the ncdns NSIS installer, remains the same.  Why publish this post anyway?  Because research isn't always as clean as people sometimes imagine it to be.  Researchers often pursue suboptimal leads; I think it's useful to document the research process authentically rather than perpetuate the myth that scientists always know what they're going to find in advance.

[1] It's not documented at [the certutil manual](https://docs.microsoft.com/en-us/windows-server/administration/windows-commands/certutil), but is mentioned elsewhere on Microsoft's website.
