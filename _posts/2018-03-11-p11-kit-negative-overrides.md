---
layout: post
title: "Negative Certificate Overrides for p11-kit"
author: Jeremy Rand
tags: [News]
---

Fedora stores its TLS certificates via a highly interesting software package called [p11-kit](https://p11-glue.github.io/p11-glue/trust-module.html).  p11-kit is designed to act as "glue" between various TLS libraries, so that (for example) Firefox, Chromium, and OpenSSL all see the same trust anchors.  p11-kit is useful from Namecoin's perspective, since it means that if we can implement Namecoin support for p11-kit, we get support for all the trust stores that p11-kit supports for free.  I've just implemented a proof-of-concept of negative Namecoin overrides for p11-kit.

As you may recall, the way our Chromium negative overrides currently work is by ~~abusing~~ utilizing HPKP such that public CA's can't sign certificates for the `.bit` TLD.  p11-kit doesn't support key pinning (it's on their roadmap though!), but there is another fun mechanism we can use to achieve a similar result: name constraints.  Name constraints are a feature of the x509 certificate specification that allows a CA certificate to specify constraints on what domain names it can issue certificates for.  There are a few standard use cases for name constraints:

1. A large organization who wants to create a lot of certificates might buy a name-constrained CA certificate from a public CA, and then use that name-constrained CA to issue more certificates for their organization.  This reduces the overhead of asking a public CA to issue a lot of certificates on-demand, and doesn't introduce any security issues because the name constraint prevents the organization from issuing certificates for domain names that they don't control.
2. A corporate intranet might create a name-constrained root CA that's only valid for domain names that are internal to the corporate intranet.  This way, employees can install the name-constrained root CA in order to access internal websites, and they don't have to worry that the IT department might be MITM'ing their connections to the public Internet.
3. A public CA might have a name constraint in their CA certificate that disallows them from issuing certificates for TLD's that have unique regulatory requirements.  For exampe, the Let's Encrypt CA [has (or at one point had) a name constraint disallowing `.mil`](https://community.letsencrypt.org/t/why-is-there-a-certificate-name-constraint-for-mil/10130), presumably because the U.S. military has their own procedures for issuing certificates.

The 1st use case is rarely ever used; I suspect that this is because it poses a risk to commercial CA's' business model.  The 2nd use case is also rarely ever used; I suspect this is because many corporate IT departments *want to* MITM all their employees' traffic, and most employees don't have much negotiating power on this topic.  But the 3rd case is quite interesting... if Let's Encrypt uses a name constraint blacklisting `.mil`, could this be used for `.bit`?

Unfortunately, we obviously can't expect all of the public CA's to have any interest in opting into a name constraint disallowing `.bit` in the way that Let's Encrypt opted into disallowing `.mil`.  However, there is a fun trick that can solve this: cross-signed certificates.  It turns out that it is possible to transform a public root CA certificate into an intermediate CA certificate, and sign that intermediate CA certificate with a root CA that we can create locally (this is called *cross-signing*).  We can then remove the original root CA from the cert store, add our local root CA and the cross-signed CA to the cert store, and everything will work just like it did before.  This is helpful because any name constraints that a CA certificate contains will apply to any CA certificate that it cross-signs.

Technically, [the RFC 5280 specification says](https://tools.ietf.org/html/rfc5280#section-4.2.1.10) that root CA's can't have name constraints.  That's not really a problem though: it just means that we have to create a local intermediate CA (signed by the local root CA) that has the name constraint, and cross-sign the public CA with the name-constrained local intermediate CA.  So the cert chain looks like this:

Local root CA (no name constraint) --> Local intermediate CA (name constraint blacklisting .bit) --> Cross-signed public CA --> (everything past here is unaffected).

Huge thanks to Crypt32 and davenpcj from Server Fault for first [cluing me in](https://serverfault.com/questions/670725/is-it-possible-to-restrict-the-use-of-a-root-certificate-to-a-domain) that this approach would work.  Unfortunately, Crypt32 didn't provide any sample code, and the sample code from davenpcj didn't work as-is for me (OpenSSL kept returning various errors when I tried to cross-sign, most of which seemed to be because OpenSSL didn't like the fact that the public CA hadn't signed an authorization for me to cross-sign their CA).  I eventually managed to cobble together a Bash script using OpenSSL that did work, but I don't think OpenSSL's command-line tool is really the right tool for the job (OpenSSL tends to rewrite large parts of the cross-signed certificate in ways that are likely to cause compatibility and security problems).  I'm probably going to rewrite this as a Go program.

Anyway, with my Bash script, I decided to apply a name constraint to `DST Root CA X3`, which is the root CA that Let's Encrypt uses.  The name constraint I applied blacklists the `.org` TLD (obviously I can't use `.bit` for testing this, since no public CA's are known to have issued a certificate for a `.bit` domain).  And... it works!  The Bash script installed the local root CA as a trust anchor for p11-kit, installed the intermediate and cross-signed CA's as trust-neutral certificates for p11-kit, and installed a copy of the original `DST Root CA X3` certificate to the p11-kit blacklist.  As a result, both Chromium and Firefox still work fine with Let's Encrypt for `.com` websites such as [Technoethical](https://tehnoetic.com/), but return an error for `.org` websites such as [Namecoin.org](https://www.namecoin.org/) -- exactly the behavior we want.

I also made a modified version of my Bash script that installs the modified CA's into a standard NSS sqlite3 database (without p11-kit), and confirmed that this works with both Firefox and Chromium on GNU/Linux.  So p11-kit probably won't be a hard dependency of this approach, meaning that this approach is likely to work for Firefox on all OS's, Chromium on all GNU/Linux distros, and anything else that uses NSS.

This code needs a lot of cleanup before it's ready for release; among the ToDos are:

* Port the certificate handling code to a Go program instead of OpenSSL's command line.
* Automatically detect which root CA's exist in p11-kit, and apply the name constraint to all of them, instead of only using `DST Root CA X3`.
* Automatically detect when a public root CA is deleted from p11-kit (e.g. WoSign), and remove the name-constrained CA that corresponds to it.
* Preserve p11-kit's attached attributes for trust anchors.
* Make the procedure idempotent.
* Test whether this works as intended for other p11-kit-supported libraries (Firefox and Chromium use NSS; p11-kit also supports OpenSSL, Java, and GnuTLS among others).
* Test whether a similar approach with name constraints can work for CryptoAPI (this would be relevant for most non-Mozilla browsers on Windows).

I'm hopeful that this work will allow us to continue supporting Chromium on GNU/Linux after Chromium removes HPKP, and that it will nicely complement the Firefox positive override support [that I added to `ncdumpzone`]({{site.baseurl}}2018/02/20/ncdumpzone-firefox.html).

It should be noted that this approach definitely will not work on most non-Mozilla macOS browsers, because macOS's TLS implementation does not support name constraints.  I'm not aware of any practical way to do negative overrides on macOS (besides the deprecated HPKP support in Chromium), so there's a chance that when we get around to macOS support, we'll just not do negative overrides for macOS (meaning that while Namecoin certificates would work on macOS without errors, malicious public CA's would still be able to do MITM attacks against macOS users just like they can for DNS domain names).  Firefox on macOS shouldn't have this problem, since Firefox doesn't use the OS for certificate verification.

This work was funded by NLnet Foundation's Internet Hardening Fund.
