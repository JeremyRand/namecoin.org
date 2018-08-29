---
layout: page
title: Enabling Your Website for Dot-Bit
---

{::options parse_block_html="true" /}

Interested in operating a Dot-Bit website? Awesome, you're in the right place.

### Namecoin-Qt

To register a Dot-Bit domain with a graphical interface (which you probably want), you first need to download Namecoin-Qt and obtain some NMC (the digital currency which runs Dot-Bit).

A command-line version, namecoind, is also provided if you want to register domains without a GUI or via scripting.

Namecoin-Qt/namecoind is primarily authored by snailbrain/thecoder/khal/vinced and is an official Namecoin project.

[You can obtain Namecoin-Qt here (download the wallet)]({{site.baseurl}}download).

### DyName

DyName is a dynamic DNS update client for Dot-Bit. It is useful if your host can switch IP addresses without warning.

Dynamic DNS is typically risky for Dot-Bit, since the decrypted keys to transfer your domain are located on an Internet-connected server.  It is therefore a good idea to use DyName in combination with [delegated alteration]({{site.baseurl}}docs/name-owners/delegated-alteration).

To obtain DyName, [visit the GitHub project.](https://github.com/JeremyRand/DyName)
