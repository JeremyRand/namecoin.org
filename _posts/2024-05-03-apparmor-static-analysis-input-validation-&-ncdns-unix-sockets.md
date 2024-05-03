---
layout: post
title: "Enhancing Security and Efficiency"
author: Robert Nganga
tags: [News]
---

Several new Electrum-NMC & Ncdns enhancements have been added.

## AppArmor Profiles for Electrum-NMC

AppArmor is a Mandatory Access Control (MAC) system which confines programs to a limited set of resources. AppArmor confinement is provided via profiles loaded into the kernel.
By using AppArmor profiles, we can ensure that Electrum-NMC operates within a predefined security policy, reducing the risk of unauthorized access or malicious actions.
The profiles are provided in the contrib folder. For more information on AppArmor, [visit their website.](https://ubuntu.com/tutorials/beginning-apparmor-profile-development#1-overview)

## Input Validation in Electrum-NMC's DNS Builder GUI
When passing data to the DNS Builder, certain validations are performed to offer hints and warnings about invalid data or outdated methods (such as algorithms). 

![IP Validation]({{ "/images/screenshots/electrum-nmc/DNS-Builder-IP-Validation-2024.png" | relative_url }})

![IPFS Images]({{ "/images/screenshots/electrum-nmc/DNS-Builder-Algorithm-Validation-2024.png" | relative_url }})

## Static Analysis Testing

We run static analysis using tools such as Flake8, MyPy, and Pylint, comparing results with Electrum (Bitcoin version) to focus on Electrum-NMC-specific issues. This process, integrated into our CI, helps catch bugs early. Looking to contribute? Static analysis issues provide good first-time issues!

## Enabling Ncdns Perfrom RPC Requests via Unix Domain Sockets

Enabling ncdns to perform RPC requests via Unix domain sockets offers a more secure communication method than TCP/IP. To utilize a Unix domain socket, specify it in the `namecoinrpcaddress` field as `unix:// + "your unix socket path"` such as `unix:///tmp/test.XXXX`

Since Namecoin Core supports only TCP/IP, we'll redirect RPC requests from the Unix Socket to Bitcoin Core. This can be achieved using a commnad such as socat command:
```
socat -d UNIX-LISTEN:"my-unix-socket-path",fork TCP:"host-address"
socat -d UNIX-LISTEN:/tmp/test.XXXX,fork TCP:localhost:8332
```
This work was funded by NLnet Foundation’s NGI Assure Fund.
