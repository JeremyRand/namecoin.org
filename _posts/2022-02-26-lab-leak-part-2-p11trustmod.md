---
layout: post
title: "The Namecoin Lab Leak (Part 2): How p11trustmod Vaccinates Against the Unmaintainable Code Omicron Variant"
author: Jeremy Rand
tags: [News]
---

In a [previous post]({{ "/2021/07/10/lab-leak-p11mod.html" | relative_url }}), I covered how splitting off p11mod from ncp11 improved code readability and auditability by using a higher-level API.  Jacob Hoffman-Andrews's p11 API is certainly more high-level than Miek Gieben's pkcs11 API, but I wasn't satisfied.  Consider that most PKCS#11 usage in the wild involves encryption or signature algorithms that operate on public or private keys.  In contrast, PKCS#11 modules like Mozilla CKBI or Namecoin ncp11 are strictly using the PKCS#11 API as a read-only database API, and only for X.509 certificates [1] -- public and private keys are nowhere to be found.  Given these limits on what usage Namecoin will need, we can construct a much higher-level (and much simpler) API than even the p11 API.

For reference, here are the function prototypes of p11:

~~~
FindObjects finds any objects in the token matching the template.
func (s Session) FindObjects(template []*pkcs11.Attribute) ([]Object, error)

Attribute gets exactly one attribute from a PKCS#11 object, returning an error if the attribute is not found, or if multiple attributes are returned. On success, it will return the value of that attribute as a slice of bytes. For attributes not present (i.e. CKR_ATTRIBUTE_TYPE_INVALID), Attribute returns a nil slice and nil error.
func (o Object) Attribute(attributeType uint) ([]byte, error)
~~~

Note that the p11 API is intentionally very generic; every object is an arbitrary mapping of attribute types (generic integers) to attribute values (generic binary data), and searching for objects requires constructing a template that looks about as generic as the object itself.  This necessarily involves a lot of boilerplate.  Here's the higher-level API, which I call p11trustmod:

~~~
type CertificateData struct {
	Label string
	Certificate *x509.Certificate
	BuiltinPolicy bool
	TrustServerAuth uint
	TrustClientAuth uint
	TrustCodeSigning uint
	TrustEmailProtection uint
}

func (b Backend) QueryCertificate(cert *x509.Certificate) ([]*CertificateData, error)
func (b Backend) QuerySubject(subject *pkix.Name) ([]*CertificateData, error)
func (b Backend) QueryIssuerSerial(issuer *pkix.Name, serial *big.Int) ([]*CertificateData, error)
func (b Backend) QueryAll() ([]*CertificateData, error)
~~~

As you can see, this API matches the behavior of libraries like NSS, which query for certificates by only a few search keys: the certificate itself, the subject, the issuer and serial, and asking for all the certificates at once.  In ncp11, we extract the `.bit` domain from the subject or the issuer.  (If the entire certificate is searched for, the subject and issuer are both within the `x509.Certificate` struct.  The `QueryAll` search method can't be used to look up `.bit` certificates, but it *can* be used to find the Namecoin root CA, which is sufficient.)  Here's some of what p11trustmod is doing behind the scenes:

* The `FindObjects` function checks which attributes are in the search template, and calls `QueryCertificate`, `QuerySubject`, or `QueryIssuerSerial` as appropriate.  Thus, ncp11 won't need to deal with processing attributes from the search template.
* The `FindObjects` function always calls `QueryAll` and appends those results.  This allows ncp11 to return certain objects such as the Namecoin root CA even if those objects aren't being referenced by name [2].
* The `FindObjects` function filters all the query results to make sure it actually matches the search template.  This allows ncp11 to simply return whatever Namecoin says matches the `.bit` domain, without worrying about the search template.
* The `Attribute` function automatically fills in sane values for boilerplate attributes such as `CKA_TOKEN` (always true), `CKA_PRIVATE` (always false), `CKA_MODIFIABLE` (always false), and `CKA_CERTIFICATE_TYPE` (always `CKC_X_509`).
* The `Attribute` function automatically extracts the subject, issuer, serial number, and hash from the certificate, so ncp11 can just return the certificate.
* The backend (e.g. ncp11) can choose whether to supply trust bits, and if so, p11trustmod synthesizes trust bits objects for the Namecoin root CA from the `BuiltinPolicy`, `TrustServerAuth`, `TrustClientAuth`, `TrustCodeSigning`, and `TrustEmailProtection` fields.  There are some tradeoffs here:
    * The `BuiltinPolicy` trust bit causes Certificate Transparency errors in Chromium, but enforces better security (e.g. blacklisting obsolete crypto and enforcing key rotations) in Firefox.
    * Omitting all trust bits from ncp11 and putting the Namecoin root CA in the standard trust store instead allows ncp11 to run with improved sandboxing.  However, depending on which trust store implementation is used, this might preclude the security benefits of the `BuiltinPolicy` trust bit.  p11-kit, which is used in Fedora and Arch, supports the `BuiltinPolicy` trust bit.  Softoken, which is used in Debian, Windows, and macOS, does not.  Since Tor Browser does not have a mutable trust store (for fingerprinting reasons), this sandboxing is probably not possible in Tor Browser.

So, all things considered, p11trustmod eliminates quite a lot of boilerplate code that ncp11 would have otherwise needed to mix in with the Namecoin-specific ncp11 code.  By proactively implementing this additional layer of abstraction between p11mod and ncp11, we reduce the risk of a second "Omicron lab leak" of insufficiently auditable code.  p11trustmod is nearly finished with implementation but still needs testing; once testing is complete, we can move on to the Namecoin-specific code in ncp11, which I expect to be quite simple since the complex things have been split out into p11mod/p11trustmod.

In other p11mod news, p11mod now supports generating key pairs.  Thanks to OpenDNSSEC for facilitating this.

This work was funded by NLnet Foundation's Internet Hardening Fund.

[1] Technically this isn't quite true.  In addition to X.509 certificates, there are a couple of other object classes that are used for setting the trust policy bits on the Namecoin root CA.  Those classes are only used by optional features, though.

[2] There are a few ways this might happen.  NSS sometimes retrieves a list of all known CA's, e.g. to display in the certificate GUI.  NSS also retrieves trust bits objects by searching for an object class or a certificate hash instead of a PKIX name or full certificate.
