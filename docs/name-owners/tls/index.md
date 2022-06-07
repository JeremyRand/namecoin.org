---
layout: page
title: Setting up TLS (for name owners)
---

{::options parse_block_html="true" /}

TLS (*Transport Layer Security*) is the "S" in HTTPS.  All websites using Namecoin should use TLS.  This guide covers how to set up TLS with Namecoin.

## Prerequisites

You should install the latest release of [the `generate_nmc_cert` tool]({{ "/download/betas/#generate_nmc_cert" | relative_url }}).  You may also want to install the latest release of the [ncdns]({{ "/docs/ncdns/" | relative_url }}) Windows installer, to test that your certificates work.

## Concepts

In Namecoin, the blockchain stores the public key of a *CA certificate* (certificate authority certificate) that you've created.  You can use that CA certificate to *issue* TLS certificates, or issue additional CA certificates (referred to as *subordinate CA certificates*).  Issuing TLS certificates or subordinate CA certificates does not require a blockchain transaction.  The CA certificate referenced by the blockchain is valid for your Namecoin domain name and all subdomains, but you can add additional restrictions when issuing TLS certificates or subordinate CA certificates.  This allows you to limit the impact of key compromise.  For example:

* You can issue TLS certificates (which will be deployed to a public-facing TLS server) with a short expiration time, while keeping the CA certificate and its private key (with a longer expiration time) on an offline machine.  You can then rotate keys by issuing a new TLS certificate periodically.  This means that if your TLS server is temporarily compromised and its private key is stolen, the situation will resolve itself the next time you rotate TLS certificates.
* You can issue TLS certificates that are only valid for certain subdomains.  For example, if you have multiple physical servers that each handle a different subset of your subdomains, you can give each of them its own TLS certificate, and a compromised server won't be able to impersonate the others.
* You can issue subordinate CA certificates that are only valid for certain subdomains.  This allows you to give a third party the ability to create TLS certificates for a specific subdomain, without letting them impact the security of the rest of your Namecoin domain.

## Example: The Basics

To create a CA certificate for your Namecoin domain, run the following:

~~~
mkdir example.bit-ca
pushd example.bit-ca
generate_nmc_cert -use-ca -use-aia -host example.bit
popd
~~~

(The directory names in these examples are arbitrary and are just intended to make the examples more clear; you can use whatever directory structure you like.)

The following files will be created in the `example.bit-ca` directory:

* `caChain.pem`: Certificate chain for issuing TLS certificates or subordinate CA's.
* `caKey.pem`: Private key for issuing TLS certificates or subordinate CA's.
* `chain.pem`: TLS certificate chain.
* `key.pem`: TLS private key.
* `namecoin.json`: TLSA record to enter in your Namecoin wallet.

(A few other files will be created too, but you don't need to worry about them.)

The JSON value contained in `namecoin.json` should be enclosed in an array, and placed in the `tls` field for the `*` subdomain of your eTLD+1 domain name.  For example:

~~~
{
    "ip": "73.239.16.12",
    "map": {
        "*": {
            "tls": [
                [
                    2,
                    1,
                    0,
                    "MDkwEwYHKoZIzj0CAQYIKoZIzj0DAQcDIgADcO7sJ5kaRYGipnz5YMXwo6NDGLWV4qVoJIn+1by1oAo="
                ]
            ]
        }
    }
}
~~~

If your Namecoin domain has no subdomains, you can just load `chain.pem` and `key.pem` into your TLS server (e.g. Caddy, Nginx, or Apache), and you're done.  That was easy.

## Example: Renewing a TLS Certificate

To issue a new TLS certificate (e.g. to rotate keys), use the `-parent-chain` and `parent-key` flags, like this:

~~~
mkdir example.bit-tls
pushd example.bit-tls
generate_nmc_cert -use-ca -host example.bit -parent-chain ../example.bit-ca/caChain.pem -parent-key ../example.bit-ca/caKey.pem 
popd
~~~

Note that `-use-ca` is still present, but `-use-aia` is not.

You'll get a new `chain.pem` and `key.pem` in the `example.bit-tls` directory, which you can load into your TLS server.

You don't need to do anything in your Namecoin wallet (or pay any fees) when renewing TLS certificates, because Namecoin TLS uses layer 2.

## Example: Issuing a TLS Certificate for a Subdomain

When using `-parent-chain` and `-parent-key`, you can enter a `-host` flag containing a subdomain, like this:

~~~
mkdir www.example.bit-tls
pushd www.example.bit-tls
generate_nmc_cert -use-ca -host www.example.bit -parent-chain ../example.bit-ca/caChain.pem -parent-key ../example.bit-ca/caKey.pem 
popd
~~~

This works as long as the `-host` flag is within the tree of the `-host` flag that the CA was created with.  For example, if the CA is for `example.bit`, the following TLS `-host` values will work fine:

* `example.bit` (same domain)
* `www.example.bit` (subdomain)
* `sub.www.example.bit` (subdomain of a subdomain)
* `www.example.bit,forum.example.bit` (multiple subdomains)
* `*.example.bit` (wildcard subdomain)
* `*.example.bit,*.*.example.bit` (multi-level wildcard)

But e.g. `example2.bit` or `example.com` will not work.

Similarly, if the CA is a subordinate CA for `www.example.bit` (see next section), the following TLS `-host` values will work fine:

* `www.example.bit`
* `sub.www.example.bit`
* `www.example.bit,sub.www.example.bit`
* `*.www.example.bit`

But the following will not work:

* `example.bit` (not within the tree of the CA)
* `www.example.bit,forum.example.bit` (all hosts must be within the tree of the CA)
* `*.example.bit`

You don't need to do anything in your Namecoin wallet (or pay any fees) when issuing TLS certificates for subdomains, because Namecoin TLS uses layer 2.

## Example: Issuing a Subordinate CA Certificate for a Subdomain

Issuing a subordinate CA certificate works like above, except you use the `-grandparent-chain` and `-grandparent-key` flags instead of `-parent-chain` and `-parent-key`, like this:

~~~
mkdir www.example.bit-ca
pushd www.example.bit-ca
generate_nmc_cert -use-ca -host www.example.bit -grandparent-chain ../example.bit-ca/caChain.pem -grandparent-key ../example.bit-ca/caKey.pem 
popd
~~~

You'll get a new `caChain.pem` and `caKey.pem` in the `www.example.bit-ca` directory, which you can use for issuing new certificates.

The `-host` flag follows similar rules as with issuing a TLS certificate.  For example, if the grandparent CA is for `example.bit`, the following subordinate CA `-host` values will work fine:

* `example.bit` (same domain)
* `www.example.bit` (subdomain)
* `sub.www.example.bit` (subdomain of a subdomain)
* `www.example.bit,forum.example.bit` (multiple subdomains)

Specifying wildcards for subordinate CA's is not necessary.

You don't need to do anything in your Namecoin wallet (or pay any fees) when issuing subordinate CA certificates, because Namecoin TLS uses layer 2.

## Testing Your Website

The best way to test your website is to try visiting it on a Windows 10 installation after running the ncdns for Windows installer.  It should load without errors.

## Selecting the Elliptic Curve

`generate_nmc_cert` defaults to the P256 curve, which balances security with compatibility.  If you really want a different curve, know what you're doing, and are prepared for the consequences, you can do so by adding one of the following flags:

* `-ecdsa-curve P224`
* `-ecdsa-curve P384`
* `-ecdsa-curve P521`
* `-ed25519`

Note that Ed25519 is more secure than the default P256, but is not widely supported by TLS implementations yet.

## Selecting the Validity Period

When issuing certificates, you can change the expiration date via the `-duration` flag, which defaults to `8760h0m0s` (365 days).  You can also use the `-start-date` flag to set the start date to something other than the present time (format is `Jan 1 15:04:05 2011`).

## Revoking Certificates

Ideally, you've picked validity periods that are short enough that you can just wait for certificates to expire when you rotate keys.  However, in the event of emergency (e.g. you have credible reason to believe that your keys have been compromised), you can revoke your keys by deleting the TLS record from the blockchain in your Namecoin wallet (which will incur a transaction fee).  You will need to start over and re-issue all TLS certificates and subordinate CA certificates.

## TLS and NS Records

Remember that if you have an NS record at or above the TLS record in the blockchain, the TLS record will be suppressed.

For example, the following configuration will **not** work:

~~~
{
    "ns": "ns21.cloudns.net.",
    "map": {
        "*": {
            "tls": [
                [
                    2,
                    1,
                    0,
                    "MDkwEwYHKoZIzj0CAQYIKoZIzj0DAQcDIgADcO7sJ5kaRYGipnz5YMXwo6NDGLWV4qVoJIn+1by1oAo="
                ]
            ]
        }
    }
}
~~~

## Cipher Suites

All TLS 1.3 cipher suites will work with Namecoin.  If you must use TLS 1.2 or lower, your TLS server will need to support an appropriate cipher suite, such as one of the `ECDHE-ECDSA` cipher suites.  If possible, please avoid using TLS 1.2 or lower (with or without Namecoin), as they are outdated and insecure.

## Using Your Own Tooling

If you prefer, you can issue subordinate CA certificates and TLS certificates using your own CA tooling.  Just point your tooling to the CA certificate and CA private key.  You should make sure that any certificates that you issue with your own tooling contain the Subject Serial Number `Namecoin TLS Certificate`.
