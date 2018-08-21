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

Dynamic DNS is typically risky for Dot-Bit, since the decrypted keys to transfer your domain are located on an Internet-connected server. DyName supports a solution to this: name importation. Register your main domain, e.g. d/valuablename, and keep its wallet encrypted. Register a throwaway name, e.g. dd/useless, and configure DyName to write an IP address to dd/useless. Use the "import" field in d/valuablename to access the IP address from dd/useless. If someone hijacks or steals the throwaway name, just register a new one and change the "import" field in your main name. If your main name has a TLS fingerprint, you won't even have a security risk if the name is stolen -- you'll just have some downtime where the site shows a TLS error.

DyName's name importation security system is fully compatible with the latest NMControl, although the older release of NMControl currently bundled with FreeSpeechMe is not compatible. FreeSpeechMe users who are using their own NMControl installation are fine.

DyName is primarily authored by Jeremy Rand AKA biolizard89.

To obtain DyName, [visit the GitHub project.](https://github.com/JeremyRand/DyName)

### Online Registrars

There are various online businesses which claim to allow registering Dot-Bit domains. While these may be slightly more convenient than downloading the Namecoin blockchain, the cost is security. The online registrar has the ability to hijack or steal your name, and so does any criminal who compromises the security of the registrar. Registrars also typically charge much higher fees; we've seen registrars charge a 50-fold markup.

A list of online Dot-Bit registrars is coming soon.

