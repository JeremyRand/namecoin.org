---
layout: post
title: "The Namecoin Lab Leak: How p11mod Vaccinates Against the Unmaintainable Code Pandemic"
author: Jeremy Rand
tags: [News]
---

Every now and then, some mad scientists will get foreign government funding, routed through an intermediary NGO, to create something in their lab, with the expectation that their creation will stay in the lab -- and then it escapes, potentially wreaking havoc.  Oops.  This is what happened to Namecoin in the months surrounding 35C3.

As those of you who watch our C3 talks will be aware, Aerth and I did some mad science in the leadup to [35C3]({{ "/2019/05/08/35c3-summary.html" | relative_url }}), in the form of ncp11.  ncp11 is a PKCS#11 module that enables Namecoin TLS certificate verification in software that uses NSS or GnuTLS (which means Firefox, the GNU/Linux release of Chromium, GNOME Web, GNU Wget, and various other software).  The ncp11 codebase that we released at our [35C3 workshop]({{ "/resources/presentations/35C3/tls-workshop/" | relative_url }}) was very much a proof of concept, as the workshop notes make clear:

> Welcome to the 35C3 Namecoin TLS Workshop, home of the code that first worked a few days ago.

We figured at the time that this was not really a big deal, since the code did work just fine, and we could always refactor and clean it up later.  And it's not like the hackers attending 35C3 have a problem with experimental code.  Alas, you already know what happened: after 35C3, we entered [major discussions with the Tor Browser developers]({{ "/2020/01/11/36c3-summary.html#lecture-adventures-and-experiments-adding-namecoin-to-tor-browser" | relative_url }}), and all of our capacity for cleaning up the ncp11 code got gobbled up.  Thus, the ncp11 codebase ended up in production use (though hopefully not in large numbers, since it was never added to the ncdns Windows installer), escaping the lab.

So, what was wrong with the ncp11 codebase that made it so in need of a refactor?  I think the best way to explain it is to examine the API it's forced to provide.  ncp11 provides an API that is equivalent to the API that [Miek Gieben's pkcs11 package](https://godocs.io/github.com/miekg/pkcs11) provides.  Here's an example consumer of this API, courtesy of Miek's documentation:

~~~
p := pkcs11.New("/usr/lib/softhsm/libsofthsm.so")
err := p.Initialize()
if err != nil {
    panic(err)
}

defer p.Destroy()
defer p.Finalize()

slots, err := p.GetSlotList(true)
if err != nil {
    panic(err)
}

session, err := p.OpenSession(slots[0], pkcs11.CKF_SERIAL_SESSION|pkcs11.CKF_RW_SESSION)
if err != nil {
    panic(err)
}
defer p.CloseSession(session)

err = p.Login(session, pkcs11.CKU_USER, "1234")
if err != nil {
    panic(err)
}
defer p.Logout(session)

p.DigestInit(session, []*pkcs11.Mechanism{pkcs11.NewMechanism(pkcs11.CKM_SHA_1, nil)})
hash, err := p.Digest(session, []byte("this is a string"))
if err != nil {
    panic(err)
}

for _, d := range hash {
        fmt.Printf("%x", d)
}
fmt.Println()
~~~

As Miek's doc prefaces this example:

> yes, pkcs#11 is verbose

This example doesn't really do us justice though, since Namecoin uses the FindObjects API, which the above example doesn't include.  Here are the function prototypes of that API:

~~~
FindObjectsInit initializes a search for token and session objects that match a template.
func (c *Ctx) FindObjectsInit(sh SessionHandle, temp []*Attribute) error

FindObjects continues a search for token and session objects that match a template, obtaining additional object handles. Calling the function repeatedly may yield additional results until an empty slice is returned. The returned boolean value is deprecated and should be ignored.
func (c *Ctx) FindObjects(sh SessionHandle, max int) ([]ObjectHandle, bool, error)

FindObjectsFinal finishes a search for token and session objects.
func (c *Ctx) FindObjectsFinal(sh SessionHandle) error

GetAttributeValue obtains the value of one or more object attributes.
func (c *Ctx) GetAttributeValue(sh SessionHandle, o ObjectHandle, a []*Attribute) ([]*Attribute, error)
~~~

Anytime you have to provide an API this low-level, the code is going to be hard to follow.  So, how to fix it?  In fact, there is actually a higher-level API available: [the p11 API by Jacob Hoffman-Andrews](https://godocs.io/github.com/miekg/pkcs11/p11), which also lives in Miek's repo (and acts as a high-level wrapper for Miek's pkcs11 API).  Here's an example consumer of Jacob's API:

~~~
module, err := p11.OpenModule("/path/to/module.so")
if err != nil {
  return err
}
slots, err := module.Slots()
if err != nil {
  return err
}
// ... find the appropriate slot, then ...
session, err := slots[0].OpenSession()
if err != nil {
  return err
}
privateKeyObject, err := session.FindObject(...)
if err != nil {
  return err
}
privateKey := p11.PrivateKey(privateKeyObject)
signature, err := privateKey.Sign(..., []byte{"hello"})
if err != nil {
  return err
}
~~~

Doesn't look as bad.  And here are the function prototypes that correspond to the above ones:

~~~
FindObjects finds any objects in the token matching the template.
func (s Session) FindObjects(template []*pkcs11.Attribute) ([]Object, error)

Attribute gets exactly one attribute from a PKCS#11 object, returning an error if the attribute is not found, or if multiple attributes are returned. On success, it will return the value of that attribute as a slice of bytes. For attributes not present (i.e. CKR_ATTRIBUTE_TYPE_INVALID), Attribute returns a nil slice and nil error.
func (o Object) Attribute(attributeType uint) ([]byte, error)
~~~

That certainly looks simpler.

So, in the same way that we created a helper library for ncp11 to use, pkcs11mod, to interface between the C ABI for PKCS#11 and the Go API for Miek's pkcs11 package, we can create a helper library (which we call p11mod) to interface between the Go API for Miek's pkcs11 package and the Go API for Jacob's p11 package.  We can then make ncp11 implement Jacob's p11 API, which allows us to make the ncp11 code substantially more readable, maintainable, and auditable.

I've now done exactly that.  p11mod has been merged as a subpackage of the pkcs11mod repository, which enables us to move forward with refactoring the Namecoin-specific parts of ncp11 (more on that in a future post).

In addition, I've improved testability of pkcs11mod/p11mod.  The repo now includes two example modules, pkcs11proxy and p11proxy.  These simply act as a passthrough shim between pkcs11mod and Miek's pkcs11 package, and p11mod and Jacob's p11 package, respectively.  Thus, you can load Mozilla's CKBI (built-in certificates) module into NSS via the pkcs11proxy and p11proxy shims, and confirm that the resulting behavior matches loading CKBI into NSS without the shims (e.g. you can do a TLS handshake and make sure that certificate acceptance behavior matches).  So, there are now nightly builds of both pkcs11proxy and p11proxy courtesy of Cirrus, as well as nightly integration tests that make sure both pkcs11proxy and p11proxy work as expected in Firefox, Chromium, GNU Wget, tstclnt, and gnutls-client, among other applications.  This is a major improvement over the previous situation, where testing pkcs11mod required Namecoin-specific tests with ncp11.

Coming soon: refactoring the Namecoin-specific parts of ncp11.
