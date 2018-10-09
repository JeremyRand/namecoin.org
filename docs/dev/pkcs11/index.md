---
layout: page
title: "PKCS11 and p11-kit Development Notes"
---

{::options parse_block_html="true" /}

This is a set of notes compiled by the Namecoin developers (in particular, Jeremy Rand) on using the pkcs11 API and/or the p11-kit software as a TLS interoperability method.  These notes were last updated on 2018 October 9.

* pkcs11  -Jeremy
    - pkcs11 specification:  -Jeremy
        + https://www.oasis-open.org/standards#pkcs11-base-v2.40  -Jeremy
    - pkcs11 allows certificates to be looked up by subject.  -Jeremy
        + NSS sometimes uses this.  https://developer.mozilla.org/en-US/docs/Mozilla/Projects/NSS/PKCS11/FAQ  -Jeremy
        + The subject of a dehydrated certificate will contain the .bit domain, so we can look it up in real-time.  -Jeremy
    - pkcs11 allows certificates to be looked up by DER value.  -Jeremy
        + NSS sometimes uses this.  https://developer.mozilla.org/en-US/docs/Mozilla/Projects/NSS/PKCS11/FAQ  -Jeremy
        + The DER value of a dehydrated certificate will contain the .bit domain, so we can look it up in real-time.  -Jeremy
    - pkcs11 allows application to ask pkcs11 smartcard to enumerate all known certificates.  -Jeremy
        + NSS sometimes uses this.  https://developer.mozilla.org/en-US/docs/Mozilla/Projects/NSS/PKCS11/FAQ  -Jeremy
        + If absolutely necessary, we can probably hook DNS lookups, and then return all dehydrated certs that were recently looked up.  We should test with NSS whether this is actually needed for our use case.  -Jeremy
    - If we want to force NSS to use a different function, we can reportedly return CKR_FUNCTION_NOT_SUPPORTED.  https://developer.mozilla.org/en-US/docs/Mozilla/Projects/NSS/PKCS11/FAQ  -Jeremy
    - pkcs11 uses the C ABI.  Articles on creating C ABI shared libraries in Go:  -Jeremy
        + Filippo Valsorda: https://blog.filippo.io/building-python-modules-with-go-1-5/  -Jeremy
        + https://id-rsa.pub/post/go15-calling-go-shared-libs-from-firefox-addon/  -Jeremy
        + https://www.darkcoding.net/software/building-shared-libraries-in-go-part-1/  -Jeremy
        + https://www.darkcoding.net/software/building-shared-libraries-in-go-part-2/  -Jeremy
        + Info on cgo:  -Jeremy
            * https://golang.org/cmd/cgo/  -Jeremy
            * https://blog.golang.org/c-go-cgo  -Jeremy
    - pkcs11 client library by Miek Gieben is at https://github.com/miekg/pkcs11 .  -Jeremy
    - Software pkcs11 modules:  -Jeremy
        + C++ SoftHSM by OpenDNSSEC is at https://github.com/opendnssec/SoftHSMv2 .  -Jeremy
        + p11-kit trust module implements a pkcs11 API provider.  -Jeremy
            * So we could use it as a base for a pkcs11 module for Namecoin cert lookups.  -Jeremy
        + NSS CKBI module also implements a pkcs11 provider.  -Jeremy
            * https://dxr.mozilla.org/mozilla-central/source/security/nss/lib/ckfw/builtins  -Jeremy
            * Looks like less volume of code than p11-kit trust module, but also appears to be way less readable than p11-kit's code.  -Jeremy
        + pkcs11-mock  -Jeremy
            * https://github.com/Pkcs11Interop/pkcs11-mock  -Jeremy
            * Windows build scripts use Visual Studio :(  -Jeremy
    - Docs on using pkcs11 with NSS: https://raymii.org/s/articles/Nitrokey_HSM_in_Apache_with_mod_nss.html  -Jeremy
    - Docs on using pkcs11 with GnuTLS: https://gnutls.org/manual/html_node/Installing-for-a-software-distribution.html  -Jeremy
    - NSS pkcs11 Module Logger: https://www-archive.mozilla.org/projects/security/pki/nss/tech-notes/tn2.html  -Jeremy
        + Works with default NSS packages in Fedora.  -Jeremy
    - Some info on Fedora pkcs11 policy (not relevant to us, since it only covers objects where the user has the private key):  -Jeremy
        + https://fedoraproject.org/wiki/User:Nmav/Pkcs11Status  -Jeremy
        + https://fedoraproject.org/wiki/Packaging:SSLCertificateHandling  -Jeremy
    - Some info on Fedora pkcs11 policy for trust anchor policy:  -Jeremy
        + https://fedoraproject.org/wiki/Features/SharedSystemCertificates  -Jeremy
    - Would p11-kit's proxy pkcs11 module be well-suited to rewriting certificates (e.g. by passing all CKBI certs through cross_sign_name_constraint_tool)?  -Jeremy
* p11-kit.  -Jeremy
    - p11-glue GitHub org:  -Jeremy
        + https://github.com/p11-glue  -Jeremy
    - p11-kit is used by default on Fedora, and can output cert databases for NSS, OpenSSL, Java, and anything that uses PEM CA bundles.  -Jeremy
    - pkcs11 support status for various applications:  -Jeremy
        + https://p11-glue.github.io/p11-glue/pkcs11-support.html  -Jeremy
    - p11-kit trust module status for Debian inclusion:  -Jeremy
        + https://lists.freedesktop.org/archives/p11-glue/2017-July/000672.html  -Jeremy
    - p11-kit includes a PEM extension that allows adding arbitrary x509 extensions to a certificate.  -Jeremy
        + Example of applying a name constraint: https://github.com/p11-glue/p11-kit/blob/master/trust/input/extensions.p11-kit  -Jeremy
        + Does this work for certificate PEM's, or only public key PEM's?  -Jeremy
            * This suggests that at least certificate PEM's work in .p11-kit.  Doesn't imply that x-certificate-extension works with certificate PEM's though.  https://github.com/p11-glue/p11-kit/blob/master/trust/persist.c#L595  -Jeremy
        + There's something called "attached extensions" (formerly called "stapled extensions") which I *think* allows storing the x509 extensions in a separate file from the PEM certificate.  https://github.com/p11-glue/p11-kit/search?q=stapled&type=Commits&utf8=%E2%9C%93  -Jeremy
            * This suggests that the stapling uses the public key as the link.  https://github.com/p11-glue/p11-kit/commit/7d4941715b5afc2ef8ea18716990d28965737c70  -Jeremy
        + Looks like attached name constraint extensions aren't currently compatible with p11-kit's crypto library retrofitting.  https://p11-glue.github.io/p11-glue/doc/storing-trust-policy/storing-trust-retrofit.html  -Jeremy
    - Registering a pkcs11 provider module (e.g. a hypothetical Namecoin certificate pkcs11 module) with p11-kit in Fedora: https://fedoraproject.org/wiki/PackagingDrafts/Pkcs11Support  -Jeremy
    - How to reset the RHEL p11-kit trust store to default settings  -Jeremy
        + https://access.redhat.com/solutions/1549003  -Jeremy
    - Does Tor Browser include libnssckbi.so?  If so, could we substitute p11-kit's drop-in replacement, and thereby make it use Namecoin via p11-kit?  -Jeremy
        + Probably would still require changes to Firefox in order to allow end-entity trust anchors.  -Jeremy
    - Mozilla bug about Firefox not using system-wide trust store.  -Jeremy
        + https://bugzilla.mozilla.org/show_bug.cgi?id=449498  -Jeremy
