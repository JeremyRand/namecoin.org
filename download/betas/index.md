---
layout: page
title: Beta Downloads
---

{::options parse_block_html="true" /}

## Warning

The downloads on this page are not yet ready for general-purpose production use.  We'd love to hear what works and what doesn't (that's why they're posted here), but don't use them in any situation where failure could result in consequences that you're unwilling to accept.  For example, don't use them with wallets that contain coins or names that you aren't willing to sacrifice to science.

The more people test these downloads, the faster they'll be ready for release.  However, there are no guarantees of when, or if, these downloads will be released in final form.

As usual, it is a good idea to verify the hashes and signatures of these downloads (especially the ones not hosted on namecoin.org).  The more people reproduced the hashes, the better.  If you're paranoid, run them inside an isolated virtual machine.

## High Priority: Lightweight SPV BitcoinJ Name Lookup Client

This is a drop-in replacement for Namecoin Core's name lookup functionality (e.g. for browsing .bit domains with ncdns), which synchronizes faster and uses less storage, but trusts Namecoin miners more than Namecoin Core does.

You need to have Java installed:

* If you're using GNU/Linux, use your package manager.
* If you're using Windows, [download it from the Oracle website](https://www.java.com/en/download/manual.jsp).  **Make sure you right-click the `.exe` installer, click `Properties`, and click `Digital Signatures`.  It should be signed by `Oracle America, Inc.`  If it is not, do not install it.**
* We're not sure about OS X.  If anyone can contribute instructions for OS X, let us know.

[Download 0.1.1 Beta 1 (bleeding-edge branch, cross-platform JAR) (Hosted by Google Drive).](https://drive.google.com/file/d/0B3JMWdAb62L5UTJQYVFVcnBKWnc/view?usp=sharing)

~~~
-----BEGIN PGP SIGNED MESSAGE-----
Hash: SHA256

bitcoinj-addons Namecoin Name Lookup Client
Version 0.1.1 Beta 1

db19dfcc23b750c799152d3ed8ff91bf67d8c64ea98992c4fe7940c75daa3514  bitcoinj-daemon-0.1.1-SNAPSHOT.jar
-----BEGIN PGP SIGNATURE-----
Version: GnuPG v2

iQIcBAEBCAAGBQJX/RRJAAoJELPy0WV4bWVw0yYP/27FaTNjPlC++yUn+7qBAIKx
RsyPJrP1wj6MvzPO3zvUdrbs1KwE1HyPs+sf31FLLRnRsal8dpg+i5ggEhXgikRm
9XE6FEuvYYmGapW/5osB1IOn7/DK37b3xQkC6+qZrG6IBxWUqQiQ4Ef4n6IejVy2
UE8Y53Ttw2i12JFaS2N+607IlaH0BNXswvNIy72TRC1zSkF8IJHcaoYvn05HeGuB
YStsuQxxZUKw71OZYuBMA5WC16WjalfqcgqxW1pDjNuoXLEyRMaSkSR4TKItfZ6p
gMDceMQe2g2HSi3sPIA5SjumQxPNgSyucg8M2YThkt5tnTQSw2m8nY4OnUBt91Hn
GRZMMu3yosPdz05lv/eQvA2BRjHYmg6pAxCjR0OUPJy6TuWT8KMPNl587lFb39vD
DxgbqkL0oPm9NEBdgHDb/cOqgSner9xAZq3eO172aFdWuWXiC+ifPjzqMrOE3XeT
QXflcy8AvJ20YyGy0DO3T4bfKBiWf7MighN5dVzylyUpoFxdgIM7cla6WZyCT3wm
mlykAL56xPxaj+WA7dux+92mDNTyo04Vm7glf9+KFwfSPVuyfZQlK9D3U0XoIyWe
3CVsKj+AGOzHM3MxgflW/4gDqiiW7IJ0LNrl0x3ksLuNUabmYi9ExgsFq3MCKWni
GpcAkajMtMUk9xJjwEax
=s/rl
-----END PGP SIGNATURE-----
~~~

[Preliminary instructions are here.]({{site.baseurl}}docs/bitcoinj-name-lookups/)

## ncdns for Windows Installer (2017-05-25)

This is an installer for ncdns for Windows which also automatically installs and configures Namecoin Core and Unbound. For more information, see the [news post announcing this release]({{site.baseurl}}2017/05/30/ncdns.html).

* [ncdns-install.exe]({{site.baseurl}}files/ncdns-2017-05-25-notreproduced/ncdns-install-2017-05-25-notreproduced.exe)
* [ncdns-install.exe.sig]({{site.baseurl}}files/ncdns-2017-05-25-notreproduced/ncdns-install-2017-05-25-notreproduced.exe.sig)
* [ncdns Installer Source Code](https://github.com/namecoin/ncdns-nsis)
* [ncdns Source Code](https://github.com/namecoin/ncdns)

### Known Issues

* Build is not yet reproducible. (This build was made by Hugo Landau.)
* The installer will prompt about setting up TLS support, but this doesn't do anything yet.

## dns-prop279

This is a tool that permits Namecoin naming (or any other naming method that speaks the DNS protocol) to be used with Tor, via the draft Prop279 pluggable naming API.  `.bit` domains can point to IP addresses (A/AAAA records), DNS names (CNAME records), and onion services.

[Source code at GitHub.](https://github.com/namecoin/dns-prop279)

### Known Issues

* Prop279 is still an early draft, and might change heavily.  dns-prop279 will change accordingly.
* `tor` doesn't implement Prop279 (see above point); the [LibreTorNS](https://github.com/namecoin/LibreTorNS) shim is required if you want to use or test dns-prop279.
* dns-prop279 doesn't follow the current Namecoin Domain Names specification for onion service records (we might amend the specification to match dns-prop279's behavior).
* dns-prop279 doesn't properly return error codes; all errors will be treated as `NXDOMAIN`.
* dns-prop279 hasn't been carefully checked for proxy leaks.
* Using dns-prop279 will make you stand out from other Tor users.
* Stream isolation isn't supported at all. Only use dns-prop279 with a DNS server that will not generate outgoing traffic when you query it. ncdns is probably fine as long as it's using a full-block-receive Namecoin node such as Namecoin Core or libdohj-namecoin in leveldbtxcache mode. Unbound is not a good idea.
* Nothing in dns-prop279 prevents the configured DNS server from caching lookups. If lookups are cached, this could be used to fingerprint users. ncdns has caching enabled by default.
* DNSSEC support hasn't been tested at all, and is probably totally unsafe right now. Only use dns-prop279 when you fully trust the configured DNS server and your network path to it.
* No binaries available yet.
* Build is not yet reproducible.
* Documentation is minimal.  Feel free to ask Jeremy if you have questions about how to use it.
