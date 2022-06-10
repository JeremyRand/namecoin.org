---
layout: post
title: "winsvcwrap: Open source replacement for SRVANY in Golang"
author: Hugo Landau
tags: [News]
---

Unlike *nix platforms, system services on Windows must be specifically designed
to run as services, and written against the Win32 Service APIs. This creates a
problem when it is desired to run a program as a Windows system service which
was not designed to function in this role.

`SRVANY.EXE` is an executable distributed by Microsoft in the Windows NT 4.0
Resource Kit, which can be used to adapt any program into a system service.
`SRVANY.EXE` is configured as the system service, and it spawns and supervises
the target process in turn. Thus, it forms an adapter between the Windows
service manager and arbitrary programs. However, `SRVANY` is a proprietary
binary for which source code is not available, and is no longer even
distributed by Microsoft.

[winsvcwrap](https://github.com/hlandau/winsvcwrap) is a simple open source Go
program to replace `SRVANY` and provide equivalent functionality, and is now
used by the Namecoin project to power its Windows installer bundle. This
enables the Namecoin project to avoid depending on an unmaintained proprietary
component. It can be used by anyone seeking to run arbitrary Windows programs
as Windows system services. Since it is written in Go, it is memory-safe
(unlike C/C++) and bootstrappable (unlike Rust).

This work was funded by Cyphrs.
