---
layout: post
title: "tstclnt: openssl s_client / gnutls-cli equivalent for NSS"
author: Jeremy Rand
tags: [News]
---

When debugging TLS handshakes, it's incredibly helpful to have a CLI tool that acts as a simple TLS client.  For OpenSSL (the TLS library used by Python, `curl`, and various other GNU/Linux things), the relevant tool is `openssl s_client`.  For GnuTLS (the TLS library used by GNOME Web, `wget`, and various other GNU/Linux things), it's `gnutls-cli`.  But did you know that there's an analogous tool for NSS (the TLS library used by Firefox and the GNU/Linux version of Chromium)?  If you didn't know this, you can be easily forgiven -- the Mozilla NSS documentation doesn't mention that it exists, and there are almost no web search results for it!  Yet it's there.

The tool is called `tstclnt` ("testclient" with the vowels removed).  On Debian Buster, you can find it in the `libnss3-tools` package.  On Fedora 34, it's a tad more complicated: `tstclnt` is in the `nss-tools` package, but it doesn't install to a directory that's on the default `PATH`.  Fedora instead installs `tstclnt` to the `/usr/lib64/nss/unsupported-tools/`   directory (path is accurate for `ppc64le`; other architectures may have slightly different paths).

Once you've installed `tstclnt`, you can run it like this:

~~~
tstclnt -b -D -h www.namecoin.org
~~~

The `-h` argument indicates which TLS server to connect to.  The `-b` flag instructs `tstclnt` to use the default CKBI (built-in certificate database) PKCS#11 module.  The `-D` flag disables the Softoken (SQLite-based certificate database) PKCS#11 module.

If all goes well, `tstclnt` will do a successful TLS handshake with `www.namecoin.org`.

For more fun, you may also wish to try the following:

* Replace `-b` with `-R /usr/lib64/pkcs11/p11-kit-trust.so` to use a non-default PKCS#11 module instead of CKBI.
* Replace `-D` with `-d sql:/etc/pki/nssdb` to use a SQLite certificate database with Softoken.  (You can use the `dbm:` prefix instead of `sql:` if you want to use Softoken's legacy BerkeleyDB database format instead of the modern SQLite.)
* Add `-C` to dump the certificate chain.  (You can use `-C -C` or `-C -C -C` for more verbosity.)
* Add `-o` to override certificate validation errors.
* Add `-p 443` to connect to a non-default TLS port.

And of course you can access a full list of options via `tstclnt --help`.

`tstclnt` is an excellent tool for TLS hackers; it's too bad Mozilla doesn't document its existence.
