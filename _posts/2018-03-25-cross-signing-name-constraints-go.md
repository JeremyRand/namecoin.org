---
layout: post
title: "Cross-Signing with Name Constraints Ported to Go"
author: Jeremy Rand
tags: [News]
---

In [my previous post about achieving negative certificate overrides using cross-signing and name constraints]({{site.baseurl}}2018/03/11/p11-kit-negative-overrides.html), I discussed how the technique could make Namecoin-authenticated TLS possible in any TLS application that uses p11-kit or NSS for its certificate storage.  However, the proof-of-concept implementation I discussed in that post was definitely not a secure implementation (nor was the code sane to look at), due to the usage of OpenSSL's command line utility (glued to itself with Bash) for performing the cross-signing.  I'm happy to report that I've ported the OpenSSL+Bash-based code to Go.

Creating a self-signed root CA in Go is relatively easy ([there's example code for it in the Go standard library](https://golang.org/src/crypto/tls/generate_cert.go), which happens to be the code on which Namecoin's `generate_nmc_cert` tool is heavily based).  Signing an existing certificate with that root CA is also pretty straightforward.  The standard library's `x509` package in Go 1.9 and higher supports creating a CA with the name constraint features we want (earlier versions of Go's `x509` package only supported a whitelist of domain names; we want a blacklist, which is newly added in 1.9).  Right now, ncdns is built with Go 1.8.3 due to a planned effort to use The Tor Project's reproducible build scripts (which currently use Go 1.8.3), but this code will be used in a standalone tool that doesn't need to be compiled into ncdns, so using Go 1.9 or higher isn't a problem.  (That said, ncdns does have pending PR's for upgrading to Go 1.9 and Go 1.10, which will be merged when The Tor Project updates their build scripts accordingly.)

The tricky part here is that Go's standard library's `x509` package does **a lot** of parsing and converting of the data in certificates, and a large amount of parsing/conversion of the CA certificate that we're cross-signing introduces an increased risk of something in the certificate being altered in a way that affects certificate validation.  This could result in unwanted errors showing up for websites (including non-Namecoin websites), which would be bad enough, but it could also result in certificates being incorrectly **accepted** -- which would mean, among other horrible things, that MITM attacks could be performed, including against non-Namecoin websites.

As one example of bad things that can happen if too much parsing is done, OpenSSL's command-line tool doesn't include x509v3 extensions by default when cross-signing.  x509v3 extensions are responsible for lots of things, including Basic Constraints (removing this would allow a CA to issue certs via intermediates that might not be valid), Name Constraints (we certainly don't want to remove all the existing name constraints when we add a` .bit`-excluding name constraint), and Key Usage and Extended Key Usage (which would make the CA valid for purposes that browsers aren't supposed to trust it for, e.g. making the CA valid for signing executable code instead of just signing TLS certificates).  While I suspect that Go is at least mildly more sane than OpenSSL's default settings, what I really wanted was a way to simply pass-through as much of the original root CA's certificate as possible when cross-signing it.

Here's the Go struct that's returned when an x509 certificate's raw DER form is passed to the `ParseCertificate` function in the standard library's `x509` package:

~~~
// A Certificate represents an X.509 certificate.
type Certificate struct {
	Raw                     []byte // Complete ASN.1 DER content (certificate, signature algorithm and signature).
	RawTBSCertificate       []byte // Certificate part of raw ASN.1 DER content.
	RawSubjectPublicKeyInfo []byte // DER encoded SubjectPublicKeyInfo.
	RawSubject              []byte // DER encoded Subject
	RawIssuer               []byte // DER encoded Issuer

	Signature          []byte
	SignatureAlgorithm SignatureAlgorithm

	PublicKeyAlgorithm PublicKeyAlgorithm
	PublicKey          interface{}

	Version             int
	SerialNumber        *big.Int
	Issuer              pkix.Name
	Subject             pkix.Name
	NotBefore, NotAfter time.Time // Validity bounds.
	KeyUsage            KeyUsage

	// Extensions contains raw X.509 extensions. When parsing certificates,
	// this can be used to extract non-critical extensions that are not
	// parsed by this package. When marshaling certificates, the Extensions
	// field is ignored, see ExtraExtensions.
	Extensions []pkix.Extension

	// ExtraExtensions contains extensions to be copied, raw, into any
	// marshaled certificates. Values override any extensions that would
	// otherwise be produced based on the other fields. The ExtraExtensions
	// field is not populated when parsing certificates, see Extensions.
	ExtraExtensions []pkix.Extension

	// UnhandledCriticalExtensions contains a list of extension IDs that
	// were not (fully) processed when parsing. Verify will fail if this
	// slice is non-empty, unless verification is delegated to an OS
	// library which understands all the critical extensions.
	//
	// Users can access these extensions using Extensions and can remove
	// elements from this slice if they believe that they have been
	// handled.
	UnhandledCriticalExtensions []asn1.ObjectIdentifier

	ExtKeyUsage        []ExtKeyUsage           // Sequence of extended key usages.
	UnknownExtKeyUsage []asn1.ObjectIdentifier // Encountered extended key usages unknown to this package.

	// BasicConstraintsValid indicates whether IsCA, MaxPathLen,
	// and MaxPathLenZero are valid.
	BasicConstraintsValid bool
	IsCA                  bool

	// MaxPathLen and MaxPathLenZero indicate the presence and
	// value of the BasicConstraints' "pathLenConstraint".
	//
	// When parsing a certificate, a positive non-zero MaxPathLen
	// means that the field was specified, -1 means it was unset,
	// and MaxPathLenZero being true mean that the field was
	// explicitly set to zero. The case of MaxPathLen==0 with MaxPathLenZero==false
	// should be treated equivalent to -1 (unset).
	//
	// When generating a certificate, an unset pathLenConstraint
	// can be requested with either MaxPathLen == -1 or using the
	// zero value for both MaxPathLen and MaxPathLenZero.
	MaxPathLen int
	// MaxPathLenZero indicates that BasicConstraintsValid==true
	// and MaxPathLen==0 should be interpreted as an actual
	// maximum path length of zero. Otherwise, that combination is
	// interpreted as MaxPathLen not being set.
	MaxPathLenZero bool

	SubjectKeyId   []byte
	AuthorityKeyId []byte

	// RFC 5280, 4.2.2.1 (Authority Information Access)
	OCSPServer            []string
	IssuingCertificateURL []string

	// Subject Alternate Name values
	DNSNames       []string
	EmailAddresses []string
	IPAddresses    []net.IP
	URIs           []*url.URL

	// Name constraints
	PermittedDNSDomainsCritical bool // if true then the name constraints are marked critical.
	PermittedDNSDomains         []string
	ExcludedDNSDomains          []string
	PermittedIPRanges           []*net.IPNet
	ExcludedIPRanges            []*net.IPNet
	PermittedEmailAddresses     []string
	ExcludedEmailAddresses      []string
	PermittedURIDomains         []string
	ExcludedURIDomains          []string

	// CRL Distribution Points
	CRLDistributionPoints []string

	PolicyIdentifiers []asn1.ObjectIdentifier
}
~~~

Holy crap, that's a lot of parsing that's clearly happening.  Among other things, we can even see special types being used for date/time values, IP addresses, and URL's.  I definitely don't want all that stuff being added to the attack surface of my cross-signing code -- it seems almost certain that some mutation would end up happening, and it would be insane to try to audit the safety of whatever mutation occurs.

However, under the hood, after the above monstrosity is serialized and is ready to be signed, a far more manageable set of structs is used by the `x509` package for the actual signing procedure:

~~~
// These structures reflect the ASN.1 structure of X.509 certificates.:

type certificate struct {
	Raw                asn1.RawContent
	TBSCertificate     tbsCertificate
	SignatureAlgorithm pkix.AlgorithmIdentifier
	SignatureValue     asn1.BitString
}

type tbsCertificate struct {
	Raw                asn1.RawContent
	Version            int `asn1:"optional,explicit,default:0,tag:0"`
	SerialNumber       *big.Int
	SignatureAlgorithm pkix.AlgorithmIdentifier
	Issuer             asn1.RawValue
	Validity           validity
	Subject            asn1.RawValue
	PublicKey          publicKeyInfo
	UniqueId           asn1.BitString   `asn1:"optional,tag:1"`
	SubjectUniqueId    asn1.BitString   `asn1:"optional,tag:2"`
	Extensions         []pkix.Extension `asn1:"optional,explicit,tag:3"`
}
~~~

Readers unfamiliar with Go programming should note that in Go, structs whose name has an uppercase first letter are public (accessible by code outside the Go standard library's `x509` package), while structs whose name has a lowercase first letter are private (only accessible from the Go standard library's `x509` package).  These more manageable structs are private, so we can't access them directly.  But, the important thing to note is that when a raw DER-encoded byte array is parsed into a crazy-complicated `Certificate`, as well as when a crazy-complicated `Certificate` is signed (resulting in a raw DER-encoded byte array), these private structs (`certificate` and `tbsCertificate`) are used as intermediate steps, and are actually processed by the `asn1` package (not the `x509` package) when converting to and from raw DER-encoded byte arrays.

The first field (`Raw`) of each struct simply holds the unparsed binary ASN.1 representation of that struct, and we can generally ignore it.  I suspect that the `tbs` in `tbsCertificate` stands for "to be signed", since it contains all of the certificate data that the signature covers (in other words, everything except the signature).  The only 5 fields that we actually will want to replace when cross-signing are the following:

* In `certificate`:
    - `SignatureAlgorithm`
    - `SignatureValue`
* In `tbsCertificate`:
    - `SerialNumber`
    - `SignatureAlgorithm`
    - `Issuer`

When a field is of type `asn1.RawValue`, it means that Go won't try to parse its content in any way (it's just a binary blob).  This is exactly the behavior we want for all the fields that we pass-through.  Unfortunately, `tbsCertificate` has a bunch of other fields besides the above 5 replaced fields that aren't of the type `asn1.RawValue`.  What can we do about that?  Well, since these are private structs to the `x509` package, we're clearly going to have to copy their definitions anyway -- so how about we simplify them while we're at it?  Below are my modified structs:

~~~
// These are modified from the x509 package; they store any field that isn't
// replaced by cross-signing as an asn1.RawValue.

type certificate struct {
	Raw                asn1.RawContent
	TBSCertificate     tbsCertificate
	SignatureAlgorithm asn1.RawValue
	SignatureValue     asn1.BitString
}

type tbsCertificate struct {
	Raw                asn1.RawContent
	Version            asn1.RawValue   `asn1:"optional,explicit,tag:0"`
	SerialNumber       *big.Int        // Replaced by cross-signing
	SignatureAlgorithm asn1.RawValue   // Replaced by cross-signing
	Issuer             asn1.RawValue   // Replaced by cross-signing
	Validity           asn1.RawValue
	Subject            asn1.RawValue
	PublicKey          asn1.RawValue
	UniqueId           asn1.RawValue   `asn1:"optional,tag:1"`
	SubjectUniqueId    asn1.RawValue   `asn1:"optional,tag:2"`
	Extensions         []asn1.RawValue `asn1:"optional,explicit,tag:3"`
}
~~~

From there, I created a modified version of the `x509` package's signing code, which creates a `tbsCertificate` by parsing the original CA certificate that we're cross-signing (instead of populating it with serialized data from the crazy-complicated `Certificate` struct as the `x509` package does), and then signs it (via ECDSA) and serializes it (via the `asn1` package) as usual.  It then outputs a DER-encoded x509 certificate that's identical to the original root CA cert, except for the 5 fields that I listed above as relevant to cross-signing.

The end result of all this work is a Go library, and associated Go command-line tool, that accepts the following as input:

* Original DER-encoded x509 certificate of a root CA to cross-sign
* A domain name to blacklist via a name constraint (defaults to `.bit`)
* Subject CommonName prefixes for the root and intermediate CA's (defaults to "Namecoin Restricted CKBI Root CA for " and "Namecoin Restricted CKBI Intermediate CA for "), which are prepended to the cross-signed CA's Subject CommonName when creating the root and intermediate CA's (the intention here is to make it easy to recognize these CA's as special based on their names)

... and outputs the following:

* DER-encoded x509 certificate of a new self-signed root CA (whose private key is destroyed)
* DER-encoded x509 certificate of a new intermediate CA (whose private key is destroyed) that has a name constraint, signed by the new root CA
* DER-encoded x509 certificate of the input root CA, cross-signed by the new intermediate CA

The root and intermediate CA's also have a Subject SerialNumber that contains the following message:

> "This certificate was created on this machine to apply a name constraint to an existing root CA.  Its private key was deleted immediately after the existing root CA was cross-signed.  For more information, see [TODO]".

(Of course, the Subject SerialNumber of the intermediate CA is also visible as the Issuer SerialNumber of the cross-signed CA.)  The intention here is that if someone encounters one of these certificates, they'll notice the SerialNumber message and therefore they won't mistakenly assume that their system has been compromised by a malicious CA certificate.  (The "[TODO]" will be replaced later by a URL that contains additional information and explains how to get the source code.)  Kudos to Ryan Castellucci for tipping me off that the Subject SerialNumber field was well-suited to this trick.

With this Go command-line tool, I applied a name constraint blacklisting `.org` to the certificate of `DST Root CA X3` (as in my previous post, this is the root CA that Let's Encrypt uses), and I got 3 new certificates as output.  I added those 3 certs to NSS's sqlite database via `certutil` (marking the new root CA as trusted), marked the old root CA as untrusted, and tried visiting the same 2 sites that use Let's Encrypt that I used in my previous post ([Technoethical](https://tehnoetic.com/) and [Namecoin.org](https://www.namecoin.org/)).  And happily, it worked just as my old OpenSSL+Bash version did: Technoethical loaded without errors (since it's in the `.com` TLD, which I didn't blacklist), but Namecoin.org showed a TLS certificate error (since it's in the `.org` TLD that I chose to blacklist).

This worked in both Chromium and Firefox on GNU/Linux.  And inspecting the resulting cross-signed certificate shows that all of the x509v3 extensions, validity period, etc. from the original `DST Root CA X3` are passed through intact.  Yay!

So, what's next?  Right now, this still takes a single root CA as input (and the user is still responsible for passing in the input root CA, and making the needed changes to NSS's database using the outputted CA's).  I've started work on a Go program that will automate this procedure for all root CA's in NSS; I'd say it's somewhere around 50% complete.  Once it's complete, this should allow us to continue supporting Chromium on GNU/Linux (even after Chromium removes HPKP), and it should also allow us to add support for Firefox on all OS's (without requiring any WebExtensions support).

This work was funded by NLnet Foundation's Internet Hardening Fund.
