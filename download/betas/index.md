---
layout: page
title: Beta Downloads
---

{::options parse_block_html="true" /}

## Warning

The downloads on this page are not yet ready for general-purpose production use.  We'd love to hear what works and what doesn't (that's why they're posted here), but don't use them in any situation where failure could result in consequences that you're unwilling to accept.  For example, don't use them with wallets that contain coins or names that you aren't willing to sacrifice to science.

The more people test these downloads, the faster they'll be ready for release.  However, there are no guarantees of when, or if, these downloads will be released in final form.

As usual, it is a good idea to verify the hashes and signatures of these downloads (especially the ones not hosted on namecoin.org).  The more people reproduced the hashes, the better.  If you're paranoid, run them inside an isolated virtual machine.

## Namecoin Core Betas

### High Priority: Name Management GUI (PR #67)

This adds a "Manage Names" tab to the GUI interface (similar to what's in Namecoin 0.3.80).  You can use it to register, track, update, and renew Namecoin names.  Testers should focus on different ways of creating and managing names, restarting the application mid-update/create/renew, and the details involved in displaying names and name transactions.

**GNU/Linux (0.12.99, April 17, 2016, commit c3cf9893c1726f7cabd8808d236be5744a76d9c2)**

Fixed issues:

* GUI rendering of Namecoin addresses was incorrect.
* No unit tests were present for pending_firstupdate wallet operations.

Known or reported issues:

* Possible wallet issues that prevent issuing of name_firstupdate and name_update transactions.
* This build failed Travis CI automated tests.  That doesn't mean that it won't work for you, but extra caution is warranted.

Downloads:

* [GNU/Linux 32-bit tar.gz (Hosted by Brandon Roberts)](http://bxroberts.org/gitian-builds/linuxbuild/namecoin-0.12.99-linux32.tar.gz)
* [GNU/Linux 64-bit tar.gz (Hosted by Brandon Roberts)](http://bxroberts.org/gitian-builds/linuxbuild/namecoin-0.12.99-linux64.tar.gz)

Gitian signatures:

Brandon Roberts

~~~
-----BEGIN PGP SIGNED MESSAGE-----
Hash: SHA256

$ bin/gbuild --url namecoin=https://github.com/brandonrobertz/namecoin-core.git --commit namecoin=c3cf9893c1726f7cabd8808d236be5744a76d9c2 ../namecoin-core-brandonrobertz/contrib/gitian-descriptors/gitian-linux.yml

13c3d8515f8ce4c6ee905b175f2f09b35a10aea2d89d355824b40223624909a0  namecoin-0.12.99-linux32.tar.gz
d6d5c5091dd4dddb0456fdd1f95f55faae16c34ba696afdc2e4ed550970ca1a0  namecoin-0.12.99-linux64.tar.gz
59d1f2130528d694fed47a8e401acf5ad076cea01a76c5b36d1c1cd830f45684  src/namecoin-0.12.99.tar.gz
e7f72914dcf1c59896cf16c65f5953970561aa6a12d55082e5be38299091f48f  namecoin-linux-0.13-res.yml
-----BEGIN PGP SIGNATURE-----
Version: GnuPG v2

iQIcBAEBCAAGBQJXJ5Z4AAoJEHk8mE2JK+fAFxcQAK6f/NjkpyEJ720plfhFfpET
Ia0qZeXs/UsdSpKTir6d4o7lvsl76GnqQVE2mP5L1sZ0CaC/2K3TI8zQKAVOt4ct
NHCnkPokRExz1cZFsi4JLGry0IebS0xrCLJ9wLCOEoljKCC/ZiZ00eMRvv4+HZy+
KhiqqrLt2XfJG3VKlRS3sOgUt1apnN04Lzu+aA6ULfZJYC4ktISxyJGMqWxnyv3c
xK8/67H4f+5Sv/iyuwSTGM8vPXBCE55LdyLobpZtTknLyH5PCSc9bHtLrUycKJjG
EUqRwfQ74yXNaOO++YjXJ8UfE/Ttnj7deZ4AVa1bWK7c/nWddBb/krbV9gxJE6x9
b3Oof+EMX2eihGRmOSxHOjMqVuJaGj4ISF6pADE78CtWojp76GGss9tLR9yIzIZi
icDIObfuA0HIjM6eBOZaF5yNx9FftChb8kDLaLvsGM94Ah+6FZnW3cdKEuEEl7Dv
NfxGEYggbzVP9YNoOULyUtoLY8x0NG96Uotyw/rr8WzhBLrbz+ThZFsOdRnD2yvr
Kac0DhBHBS4/KJ9QfRNGP98Ch6ATMtfjtLmNdK5idT9sqQDC8cZKwap2BzIVnBLR
qU0sKaWnjuzGwtOIpoYzMXAbHxJ5D9kkA/dOOXcmf7YWNF5rqd2ZRwsQ0HggU/Im
dcMTUgcAJWp1bxNDLvGp
=iJ1x
-----END PGP SIGNATURE-----
~~~

**Windows (0.12.99, April 17, 2016, commit 4de287f606f3fc156839ee8fe0309f3b6f7e8529)**

Fixed issues:

* GUI rendering of Namecoin addresses was incorrect.
* No unit tests were present for pending_firstupdate wallet operations.

Known or reported issues:

* Possible wallet issues that prevent issuing of name_firstupdate and name_update transactions.
* A temporary patch is applied for Windows builds that doesn't match Bitcoin Core's code.
* This build failed Travis CI automated tests.  That doesn't mean that it won't work for you, but extra caution is warranted.

Downloads:

* [Windows 32-bit Installer (Hosted by Brandon Roberts)](http://bxroberts.org/gitian-builds/winbuild/namecoin-0.12.99-win32-setup-unsigned.exe)
* [Windows 32-bit zip (Hosted by Brandon Roberts)](http://bxroberts.org/gitian-builds/winbuild/namecoin-0.12.99-win32.zip)
* [Windows 64-bit Installer (Hosted by Brandon Roberts)](http://bxroberts.org/gitian-builds/winbuild/namecoin-0.12.99-win64-setup-unsigned.exe)
* [Windows 64-bit zip (Hosted by Brandon Roberts)](http://bxroberts.org/gitian-builds/winbuild/namecoin-0.12.99-win64.zip)

Gitian signatures:

Brandon Roberts

~~~
-----BEGIN PGP SIGNED MESSAGE-----
Hash: SHA1

$ bin/gbuild --url namecoin=https://github.com/brandonrobertz/namecoin-core.git --commit namecoin=4de287f606f3fc156839ee8fe0309f3b6f7e8529 ../namecoin-core-brandonrobertz/contrib/gitian-descriptors/gitian-win.yml

2fa74227e4fd1341788a8688c4f1d64f9ef913f1f22cbce5b23c8aab9aedac88  namecoin-0.12.99-win-unsigned.tar.gz
edfcbcb9762c6464a5dee6195e12b146dad5439a3603fe7e2acc0ff970305525  namecoin-0.12.99-win32-setup-unsigned.exe
f4557f8ded3bdbce121f0293af09772f6151f16daaac14e3f2417e0935f2c993  namecoin-0.12.99-win32.zip
4ebcbace05109081a84b311c228f4e4f3653c054aa6b63de2e7e2c1687dc7595  namecoin-0.12.99-win64-setup-unsigned.exe
6cb88d52934a51abde1e75d38ea6ea24d25f2ee68c4cfe180624f797e17cedc0  namecoin-0.12.99-win64.zip
38205f3dbd6405a4546e2eb8918cc30541d4134ef5de8c6d11e36ee6ab59255d  src/namecoin-0.12.99.tar.gz
c1df98f012b35a395b29ee7dd87f77587abcf3a22e9bc0d3e46d3b6ffdab78bf  namecoin-win-0.13-res.yml
-----BEGIN PGP SIGNATURE-----
Version: GnuPG v1

iQIcBAEBAgAGBQJXJqfyAAoJEHk8mE2JK+fAVbcP/3dWOI+/8GUEMJJmAirAQ9Np
9lFk2aUhgM9fssz3vPVpGmhGX2OxcBkVPxlCobjff0P2n1AGCRUwZgvm8lxqa1XJ
0zkOMBcsQPJaZLo7mlWprxoj8dmjPGSQ66lBd1ekqjIyZScRJKK8Zux/3FWyaPjD
HVdT68WzYI6jJOuhGz8SjvnBbOTXYpty7WxGtPASNu8GNQ+EisKBcj9Nbh70l1bD
UZaabgpb/9YUrIYF3+LGQB5Mav+XVV5JM+eAYN+NKle257zHAHolQxKcGzybzRAn
TgTQZaDdBPOO8SotpxVpahZ/R0YzFuszet+XMowI9m3d/fVogmqMOkZYv8thulRN
/jPMRvFCxqeZeRlFDFn+AJ2wyTocOPV5ggtl+30mkqWyGaW8wfgvco6+BWweHLml
c4GM0IZAedmhspEPA+BiE8B1wX/XujK4aXmxtd4wCOaTjDRKTVt0BFtu5rX0kzE5
PAXH5+mjs9Eze+VJcJX171jqSEHa/sAGkCK/WjHjEPTG0XjC5jLG6vP2BXr33YNG
T2PFzcxpp+Ah4gVrwhgNUPj/38vjy3UN544u8WQBXPPwXMDB810qyJxrCCLxN/6S
Z57yOIO7Zn3oyQG5wSEAkHKheO+O/auPosUcYSIl4bmZiARdEfaP43njBXrSYLmn
Jqrrvt1s7whLLy/B1lDB
=DWZU
-----END PGP SIGNATURE-----
~~~

**Windows (0.12.99, April 16, 2016, commit f7b2996c5bd7ed8dc50a1be730a4c9979039a94c)**

Known or reported issues:

* Possible wallet issues that prevent issuing of name_firstupdate and name_update transactions.
* GUI rendering of Namecoin addresses is incorrect.
* No unit tests for pending_firstupdate wallet operations.
* A temporary patch is applied for Windows builds that doesn't match Bitcoin Core's code.
* This build failed Travis CI automated tests.  That doesn't mean that it won't work for you, but extra caution is warranted.

Downloads:

* [Windows 32-bit zip (Hosted by Google Drive)](https://drive.google.com/open?id=0B6-yIQJRtCE-azgzcnJvbENnTDg)
* [Windows 64-bit zip (Hosted by Google Drive)](https://drive.google.com/open?id=0B6-yIQJRtCE-YU8tSVRQSUZXNGc)

Gitian signatures:

Brandon Roberts

~~~
-----BEGIN PGP SIGNED MESSAGE-----
Hash: SHA256

$ bin/gbuild --url namecoin=https://github.com/brandonrobertz/namecoin-core.git --commit namecoin=f7b2996c5bd7ed8dc50a1be730a4c9979039a94c ../namecoin-core-brandonrobertz/contrib/gitian-descriptors/gitian-win.yml

7e6e2f91b88fe62f7759254b22cdacc59c18fd5c3f5da7c44aa41e408d427c36  namecoin-0.12.99-win-unsigned.tar.gz
aea1ddc4bbe1b34e5e8456b63bffb5c101f44b197c03ef76149f94f1225a4bb5  namecoin-0.12.99-win32-setup-unsigned.exe
3836bef4777ebd5ec0b0a0addaedba611689c73003006eae1e2ed0ecd1deda8a  namecoin-0.12.99-win32.zip
d07238afefcd95bdac18b73bb6d5c2fa40f3427652aa19a64e15d1edd52abb34  namecoin-0.12.99-win64-setup-unsigned.exe
fe044a222d58be59efb84a168b6f4e2a584dd76b75beef0a3d9a22ea66d7be4c  namecoin-0.12.99-win64.zip
e78621c3d99db314587a3f26b2b9f5d056d94f2eae477995e87e33e2b42f0a03  src/namecoin-0.12.99.tar.gz
0ee8ea8043bceb0fc56e129bf63f1fd5862d6e60f663a1bbf184a64a01b0bb40  namecoin-win-0.13-res.yml
-----BEGIN PGP SIGNATURE-----
Version: GnuPG v2

iQIcBAEBCAAGBQJXEurQAAoJEHk8mE2JK+fAgIgP/A0R2/AweGmaMn6CFZq2EYVy
JvVOwUh1bAjss5hqNojfdazLlsU7NHVTEXP5DydaMI9Vlpm3wgcwrUUDrA3z/p7w
F44anUuz5OpGUYg1kofUK3Fos/GlrkNYRdmMCj9dPw2LR+DXbwr4PPs42OGGTdpN
/2PnUeN3zTrLjOD4DMUAvH5XDAdJzxidZPrYGNv+o4l5+K8hSFXzjddu4Az50RZ/
4d0yKL2fDABFN5lUABQnTSDkcv2rMtlARH+9N0nDkapBPTyQdEFs7Yc4SARl6w4D
RZIUNgOcmAhkzamO5Q2/kgUtI/kNusrUAMi73eUuo7qPql0ryuzXGOAHS25QGbvy
w6zVqhnwxkGfwMjCctp9hqAFPJqDZUCdQD38OkrLd3ospuQFn6sBGSH/9ur59kQW
n9PEu/vEzLmzVKD/CNk4d+Ym5suLk3kzxwnPLIn0mgA7zSePHTRMHjnLi1YfIHql
ADbG+66Tw708wu5WoaHNR5n755xZT59MVr0F30/y3u0LkRf3u4YB5M+41IZSsXT9
apKFgGq5z4ovgCccF67Eg+BNZKXWqBAYUv8O7SyPte49EAW/McM6XG2h0/SwvL5Q
xHNgqoNEDao8zeF/XsKpAlkzI2ekEsIfQYEIc0+EQjf9sSvTntVlEYsBmTTsbaty
qHv3sB8poBECWq9HjjCk
=tTuR
-----END PGP SIGNATURE-----
~~~

Joseph Bisch

~~~
-----BEGIN PGP SIGNED MESSAGE-----
Hash: SHA256

$ bin/gbuild --url namecoin=https://github.com/brandonrobertz/namecoin-core.git --commit namecoin=f7b2996c5bd7ed8dc50a1be730a4c9979039a94c ../namecoin-core-brandonrobertz/contrib/gitian-descriptors/gitian-win.yml

7e6e2f91b88fe62f7759254b22cdacc59c18fd5c3f5da7c44aa41e408d427c36  namecoin-0.12.99-win-unsigned.tar.gz
aea1ddc4bbe1b34e5e8456b63bffb5c101f44b197c03ef76149f94f1225a4bb5  namecoin-0.12.99-win32-setup-unsigned.exe
3836bef4777ebd5ec0b0a0addaedba611689c73003006eae1e2ed0ecd1deda8a  namecoin-0.12.99-win32.zip
d07238afefcd95bdac18b73bb6d5c2fa40f3427652aa19a64e15d1edd52abb34  namecoin-0.12.99-win64-setup-unsigned.exe
fe044a222d58be59efb84a168b6f4e2a584dd76b75beef0a3d9a22ea66d7be4c  namecoin-0.12.99-win64.zip
e78621c3d99db314587a3f26b2b9f5d056d94f2eae477995e87e33e2b42f0a03  src/namecoin-0.12.99.tar.gz
03909f7d1ac040766e6f82ad222c4cb4be79dffd5dcb43e7c01c4ee124fa8146  namecoin-win-0.13-res.yml
-----BEGIN PGP SIGNATURE-----
Version: GnuPG v2

iQIcBAEBCAAGBQJXE+nkAAoJEM8I+sM5q36O4LAP/i9V5KwCLzmbL9zeDd1swgdu
6TvOSMtXYksu5Nlv32KlYOyYzyZwc+5JCXlHNIGOimO9yHfuepJgUQT5axi1y50i
sEUTpoWxtZD9FGbfih4TbKseiu1URe4xR2GO2ypR9BbY/mK29Kg3g7r0pmY9TktS
sSSi1SaNpq8yOPjBOrI7f+i2KQcy5VSByA023UdFUTk1pigP71qjZGOw+OQeG8Bp
CAW7hNvCj6WpKArUJuAH7fs+wL5+nebiuVfzEkqNvOp1m6iebXQv6Jvtt2eufEB+
Pd53wvRcC6ndFfSrpUfZiiPkRy71RMMv7wdkGBJP5VAqtcIKGOJyaSvjpZZVY8vG
AJnZEiU8kvv7SrHo59J/9PO4HAVsLKkBNuf5mHoT9vj6EhPXv7unSsZelQzLxu2N
/rInZ5WzBfMwko58m0/dgUy2v6HhKzVkCjCJvFAcQDe5NNbDHeHMdqvALghTig2x
Fh4mSXxzk3ZGZcSBQ6a3+6u1ORbfAQGSUA1JD4RFErxuenVlp8SvoEIrZ1y1WEuK
3mDXUIShF4lAELI6wHeFO6zsomy+CQMdMATPLg+tqNAOC6c2P93nFeP6w58maNo/
HhiiW/JIZ6LsoQSHMpM3D/l6gNRpG63HdcQJ8XL0LaxW7rMmCZ/hVBiquMZ7y7nF
Y8KZVgiGjFAqQ45j4ngB
=iBUs
-----END PGP SIGNATURE-----
~~~
