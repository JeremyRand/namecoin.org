---
layout: page
title: Download
---

{::options parse_block_html="true" /}

## ncdns

ncdns is software for accessing `.bit` domain names.  If you want to access `.bit` domain names, ncdns is most likely what you want to install.

See the [ncdns documentation]({{ "/docs/ncdns" | relative_url }}).

The ncdns Windows installer also automatically installs and configures a Namecoin client (Namecoin Core, ConsensusJ-Namecoin, or Electrum-NMC) and Dnssec-Trigger/Unbound, and sets up TLS certificate validation in any supported web browsers that are installed (see documentation for a list of supported browsers).  It's basically all you need for browsing `.bit` domain names.

**Before running the ncdns Windows installer, you will need to install the following:**

* [Visual C++ Redistributable for Visual Studio 2015](https://www.microsoft.com/en-us/download/details.aspx?id=53587)

ncdns plain binaries are also available for most major operating systems.  These are useful for advanced users or for users who are not on Windows.  Using these will require setting up a Namecoin client (Namecoin Core, ConsensusJ-Namecoin, or Electrum-NMC) and a recursive DNS resolver (e.g. Unbound) separately; they can sometimes be used for TLS certificate validation, but additional setup is required.

Current release: 0.3.

* [ncdns v0.3 GNU/Linux (64-bit x86) plain binaries]({{ site.files_url }}/files/ncdns/ncdns-0.3/ncdns-0.3-linux-x86_64-e41ca2.tar.xz)
* [ncdns v0.3 GNU/Linux (32-bit x86) plain binaries]({{ site.files_url }}/files/ncdns/ncdns-0.3/ncdns-0.3-linux-i686-990513.tar.xz)
* [ncdns v0.3 Windows (64-bit x86) installer]({{ site.files_url }}/files/ncdns/ncdns-0.3/ncdns-0.3-x86_64-install-a1e8b8.exe)
* [ncdns v0.3 Windows (64-bit x86) plain binaries]({{ site.files_url }}/files/ncdns/ncdns-0.3/ncdns-0.3-windows-x86_64-ac643c.tar.xz)
* [ncdns v0.3 Windows (32-bit x86) plain binaries]({{ site.files_url }}/files/ncdns/ncdns-0.3/ncdns-0.3-windows-i686-e076b4.tar.xz)
* [ncdns v0.3 macOS (64-bit x86) plain binaries]({{ site.files_url }}/files/ncdns/ncdns-0.3/ncdns-0.3-osx-x86_64-b17766.tar.xz)
* [ncdns v0.3 Hashes]({{ site.files_url }}/files/ncdns/ncdns-0.3/sha256sums-unsigned-build.txt)
* [ncdns v0.3 Signature (Release signed by Jeremy Rand)]({{ site.files_url }}/files/ncdns/ncdns-0.3/sha256sums-unsigned-build.txt.asc)
* [ncdns Windows Installer Source Code](https://github.com/namecoin/ncdns-nsis)
* [ncdns rbm Build Harness Source Code](https://github.com/namecoin/ncdns-repro)
* [ncdns Source Code](https://github.com/namecoin/ncdns)

## Namecoin Core Client (Stable Release)

**If you installed the ncdns Windows installer, then you probably already have Namecoin Core.**

* Name wallet: includes command-line interface for registering, tracking, updating, and renewing names (if you don't already have some namecoins, you'll need to [buy some at an exchange]({{ "/exchanges/" | relative_url }})).
* **No graphical interface for name wallet.  Use the Name Tab Beta (see below) if you require this functionality.**
* Name lookup: allows looking up names (use in combination with [ncdns]({{ "/docs/ncdns" | relative_url }}) or NMControl to browse `.bit` domains).
* Currency wallet: includes graphical interface and command-line interface for receiving and sending namecoins.
* Recommended for miners.
* Recommended for users who don't own any names.
* Recommended for users who are comfortable registering, updating, and renewing their names via the command-line.

Current release: 0.21.0.1.

* [Namecoin Core 0.21.0.1 (GNU/Linux RISC-V 64-bit)]({{ site.files_url }}/files/namecoin-core/namecoin-core-0.21.0.1/namecoin-nc0.21.0.1-riscv64-linux-gnu.tar.gz)
* [Namecoin Core 0.21.0.1 (GNU/Linux ARM 64-bit)]({{ site.files_url }}/files/namecoin-core/namecoin-core-0.21.0.1/namecoin-nc0.21.0.1-aarch64-linux-gnu.tar.gz)
* [Namecoin Core 0.21.0.1 (GNU/Linux ARM 32-bit)]({{ site.files_url }}/files/namecoin-core/namecoin-core-0.21.0.1/namecoin-nc0.21.0.1-arm-linux-gnueabihf.tar.gz)
* [Namecoin Core 0.21.0.1 (GNU/Linux x86 64-bit)]({{ site.files_url }}/files/namecoin-core/namecoin-core-0.21.0.1/namecoin-nc0.21.0.1-x86_64-linux-gnu.tar.gz)
* [Namecoin Core 0.21.0.1 (Windows 64-bit installer)]({{ site.files_url }}/files/namecoin-core/namecoin-core-0.21.0.1/namecoin-nc0.21.0.1-win64-setup-unsigned.exe)
* [Namecoin Core 0.21.0.1 (Windows 64-bit zip)]({{ site.files_url }}/files/namecoin-core/namecoin-core-0.21.0.1/namecoin-nc0.21.0.1-win64.zip)
* [Namecoin Core 0.21.0.1 (macOS dmg)]({{ site.files_url }}/files/namecoin-core/namecoin-core-0.21.0.1/namecoin-nc0.21.0.1-osx-unsigned.dmg)
* [Namecoin Core 0.21.0.1 (macOS tar.gz)]({{ site.files_url }}/files/namecoin-core/namecoin-core-0.21.0.1/namecoin-nc0.21.0.1-osx64.tar.gz)
* [Namecoin Core 0.21.0.1 (Source code tarball)]({{ site.files_url }}/files/namecoin-core/namecoin-core-0.21.0.1/namecoin-nc0.21.0.1.tar.gz)
* [Namecoin Core Gitian signatures](https://github.com/namecoin/gitian.sigs/)
* [Namecoin Core source code](https://github.com/namecoin/namecoin-core/)

## Namecoin Core Client (with Qt Name Tab)

* Name wallet: includes graphical interface and command-line interface for registering, tracking, updating, and renewing names (if you don't already have some namecoins, you'll need to [buy some at an exchange]({{ "/exchanges/" | relative_url }})).
* Name lookup: allows looking up names (use in combination with ncdns or NMControl to browse .bit domains).
* Currency wallet: includes graphical interface and command-line interface for receiving and sending namecoins.
* **Not suitable for mining.**

Current release: 0.13.99-name-tab-beta1.

[Source on GitHub](https://github.com/namecoin/namecoin-core).

<div class="row">

<div class="col-sm-4">

### Windows

* [64-bit Installer]({{ site.files_url }}/files/namecoin-core/namecoin-core-0.13.99-name-tab-beta1-notreproduced/namecoin-0.13.99-win64-setup-unsigned.exe)
* [32-bit Installer]({{ site.files_url }}/files/namecoin-core/namecoin-core-0.13.99-name-tab-beta1-notreproduced/namecoin-0.13.99-win32-setup-unsigned.exe)
* [64-bit zip]({{ site.files_url }}/files/namecoin-core/namecoin-core-0.13.99-name-tab-beta1-notreproduced/namecoin-0.13.99-win64.zip)
* [32-bit zip]({{ site.files_url }}/files/namecoin-core/namecoin-core-0.13.99-name-tab-beta1-notreproduced/namecoin-0.13.99-win32.zip)

[Gitian signatures at GitHub.](https://github.com/namecoin/gitian.sigs/tree/master/0.13.99-name-tab-beta1-win-unsigned)

</div>

<div class="col-sm-4">

### GNU/Linux

* [x86 64-bit tgz]({{ site.files_url }}/files/namecoin-core/namecoin-core-0.13.99-name-tab-beta1-notreproduced/namecoin-0.13.99-x86_64-linux-gnu.tar.gz)
* [x86 32-bit tgz]({{ site.files_url }}/files/namecoin-core/namecoin-core-0.13.99-name-tab-beta1-notreproduced/namecoin-0.13.99-i686-pc-linux-gnu.tar.gz)
* [ARM 64-bit tgz]({{ site.files_url }}/files/namecoin-core/namecoin-core-0.13.99-name-tab-beta1-notreproduced/namecoin-0.13.99-aarch64-linux-gnu.tar.gz)
* [ARM 32-bit tgz]({{ site.files_url }}/files/namecoin-core/namecoin-core-0.13.99-name-tab-beta1-notreproduced/namecoin-0.13.99-arm-linux-gnueabihf.tar.gz)

[Gitian signatures at GitHub.](https://github.com/namecoin/gitian.sigs/tree/master/0.13.99-name-tab-beta1-linux)

</div>

<div class="col-sm-4">

### macOS

**macOS binaries of Namecoin Core Name Tab Beta are not yet available.  For now, macOS users should use the Stable Release (see above), build Namecoin Core from source, or use Namecoin Legacy (see below).**

</div>

</div>

## Namecoin Legacy Client

* Use this instead of Namecoin Core if Namecoin Core doesn't work or if you need macOS binaries that include the name management tab.
* **Not suitable for mining or development.**
* **Always wait for six confirmations.**
* **Backup your `wallet.dat` before enabling wallet encryption.**

[Source on GitHub](https://github.com/namecoin/namecoin-legacy).

<div class="row">

<div class="col-sm-4">

### Windows

Download [v0.3.80 installer]({{ site.files_url }}/files/namecoin-core/namecoin-core-0.3.80/Namecoin_v0.3.80_setup.exe)<br>
[Release information and signatures]({{ site.forum_url }}/viewtopic.php?f=8&t=2123).

</div>

<div class="col-sm-4">

### GNU/Linux Packages

GNU/Linux packages provided by OBS.<br>
[Command line client (namecoind)](https://software.opensuse.org/download.html?project=home%3Ap_conrad%3Acoins&amp;package=namecoin).<br>
[Graphical interface (Namecoin-Qt)](https://software.opensuse.org/download.html?project=home%3Ap_conrad%3Acoins&amp;package=namecoin-gui).

</div>

<div class="col-sm-4">

### macOS

Download [v0.3.80 Namecoin-Qt]({{ site.files_url }}/files/namecoin-core/namecoin-core-0.3.80/Namecoin-Qt.app-0.3.80-a00c33d.zip).<br>
[Release information]({{ site.forum_url }}/viewtopic.php?f=8&t=2235).

</div>

</div>

## NMControl Middleware

NMControl connects your browser/application to the client. It allows you, for example, to browse .bit domains. [Source and installation instructions on Github](https://github.com/namecoin/nmcontrol).

**Generally, we recommend ncdns instead of NMControl.**  Usage of NMControl may be warranted in the following circumstances:

* You rely on reproducible builds of the non-installer binaries.  (Python source code is inherently reproducible; ncdns is not yet reproducible.)
* You rely on the non-default Private Mode of NMControl.  (Private Mode prevents external DNS traffic from `.bit` lookups; ncdns can trigger such lookups from Unbound.)
* You rely on encrypted DNS (e.g. DoH) for non-`.bit` lookups, **and** you rely on the Windows installer (NMControl's installer won't affect non-`.bit` DNS lookups, while ncdns installs DNSSEC-Trigger, which will interfere with DoH.)

<div class="row">

<div class="col-sm-4">

### Windows

Download [v0.8.1 installer]({{ site.files_url }}/files/nmcontrol/nmcontrol-0.8.1/NMControl_v0.8.1_setup.exe).<br>
[Release information and signatures (read this first)]({{ site.forum_url }}/viewtopic.php?f=8&t=2402).

</div>

<div class="col-sm-4">

### GNU/Linux and macOS

With Python 2.7 installed you can run nmcontrol.py directly from [source](https://github.com/namecoin/nmcontrol). If you need help ask on Github or on the [forum]({{ site.forum_url }}/viewtopic.php?f=8&t=2402).

</div>

</div>

## Help Us Test Betas

Want to help us test improvements in Namecoin?  Check out the [Beta Downloads]({{ "/download/betas/" | relative_url }}) page.
