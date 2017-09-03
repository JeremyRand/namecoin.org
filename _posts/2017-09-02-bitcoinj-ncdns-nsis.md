---
layout: post
title: BitcoinJ support merged into ncdns-nsis
author: Hugo Landau
tags: [News]
---

The ncdns-nsis project, which provides a zero-configuration Windows installer
for Namecoin domain name resolution functionality, has merged SPV support,
implemented via BitcoinJ. This enables Windows machines to resolve `.bit`
domain names without having to download the full Namecoin chain.

Merging this functionality means that Namecoin domain names can be resolved on
Windows client machines with only a minimal chain synchronization and storage
burden, in exchange for a limited reduction in the security level provided.

Installer binaries will be published in due course as remaining ncdns-nsis
issues are concluded.

To use the BitcoinJ client, run the ncdns-nsis installer and select the SPV
option when asked whether to install a Namecoin node. Run BitcoinJ after
installation and wait for synchronization to complete; `.bit` name resolution
is then available.
