---
layout: post
title: "Enhancements to IPFS & IPNS: A Closer Look at the Latest Updates"
author: Robert Nganga
tags: [News]
---


## The Dedicated GUI for IPFS & IPNS

Previously, interacting with IPFS required the creation of TXT records and knowledge of IPFS DNSLink. However, with the introduction of a dedicated GUI, users now have a more user-friendly option, available in the address tab. To learn more about DNSLink and its use of DNS TXT records to map names to IPFS addresses or IPNS names, refer to the [IPFS DNS documentation.](https://docs.ipfs.tech/concepts/dnslink/)

![IPFS Images]({{ "/images/screenshots/electrum-nmc/IPFS-GUI-Introduction.png" | relative_url }})

<video controls>
<source src="{{ site.files_url }}/files/videos/docs/ipfs/brave-ipfs.webm" type="video/webm">
Demo video of Namecoin + IPFS in Brave.
</video>

## Added Option for Value Encoding

When registering names, there is now an option to pass values as binary, encoded in hexadecimal format. This enhancement provides greater flexibility and control for developers and power users, allowing them to represent domain name values in a more efficient and versatile manner.

![Binary Encoding (value)]({{ "/images/screenshots/electrum-nmc/Binary-Encoding-Values-GUI.png" | relative_url }})

## GUI Testing

For GUI testing, we validate the GUI form's build process and execute regression tests, which helps us identify any errors in the files.

## Windows Binaries Testing

We validate the integrity of the build process and conduct regression tests on these binaries to ensure their functionality and detect any potential issues.

This work was funded by NLnet Foundation’s NGI Assure Fund.
