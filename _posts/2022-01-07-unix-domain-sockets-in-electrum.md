---
layout: post
title: "Unix domain sockets in Electrum"
author: yanmaani
tags: [News]
---

I recently sent in a patch to Electrum which adds support for Unix domain sockets to the RPC interface. After review by ghost43 and Jeremy Rand, who is also a Namecoin developer, this patch has been merged. As a result, Electrum now has support for Unix domain sockets in the RPC daemon. (Unix domain sockets are not yet supported for other occasionally local network operations, such as for connecting to a SOCKS proxy.)

Unix domain sockets are a type of sockets that run entirely inside of the system (the "Unix domain"). Instead of binding to a network port, they bind to a file. The access to this file can then be controlled by the ordinary mechanisms, like file system permissions or ACLs. This makes them very well-suited to replace connections to the loopback interface, because unlike such connections, they are not visible to the entire system. Moreover, by completely disabling the ability of an application to make network requests and limiting its connections to a few select Unix domain sockets, a very high degree of control can be maintained.

It is envisioned that this will be useful for `ncdns`, our DNS resolver, because it will allow Electrum-NMC to provide an RPC interface to `ncdns` without having to bind to a network port. This reduces attack surface, making the application more secure. The functionality can also be used by other Electrum users for other use-cases, and some are even suggesting to make it the default on Unix platforms.

I would like to thank Jeremy Rand and ghost43 for reviewing this pull request, pointing out several important details I missed and giving me helpful feedback on future-proofing and codebase standards.

This change does not in any way impact how Electrum makes connections to other hosts over the network. This change is not expected to break any existing behaviors.

This work was funded by NLnet Foundation's NGI0 Discovery Fund.
