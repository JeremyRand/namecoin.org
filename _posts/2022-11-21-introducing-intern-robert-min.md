---
layout: post
title: "Introducing Namecoin's New Intern, Robert Nganga"
author: "Jeremy Rand and Robert Nganga"
tags: [News]
---

Hey everyone, Jeremy here.  We're happy to announce that Namecoin has accepted an intern via [Outreachy](https://www.outreachy.org/) (we are participating under the Tor Project umbrella).  Robert Nganga will be funded to work on a new project that I think both the Namecoin and Tor communities will find exciting.  Huge thanks are due to Tor Project for taking us under their wing for this internship.  I don't want to steal his thunder any more than I already have, so I'm going to turn over the remainder of this post to Robert to tell you about his project -- please join us in welcoming him to the Namecoin community!

Greetings, I am pleased to join the Namecoin team as a [Outreachy](https://www.outreachy.org/) Intern after two months of contributing to Namecoin. I'll be working on developing a tool that can identify and prevent proxy leaks. I'll be using ptrace to intercept network socket system calls in a Tor-using application and block them if they constitute a proxy leak. Given the substantial threats posed by proxy leaks, the technology will improve security and privacy while resolving current problems with the technologies already in use. For instance, torsocks, an LD_PRELOAD-based program that intercepts calls to the network functions of the standard C library, is incompatible with statically linked software, such as the majority of Golang-written applications (Given that Golang is frequently used for security-conscious applications, this is particularly problematic) Transparent proxy tools like Whonix can result in usage of multiple applications being traceable to the same user, since you can't easily enforce stream isolation (degrading anonymity into much weaker pseudonymity)

Furthermore, prior Namecoin development such as the Namecoin support in Tor Browser, was for a period slowed down by manual audits for proxy leaks. In the future, the tool will prevent such bottlenecks. This ptrace approach has a [crude proof-of-concept](https://github.com/JeremyRand/heteronculous) in Bash, which was developed by the Namecoin team.

The project will be mentored by Jeremy Rand, who will also provide any support required.

This work was funded by Outreachy under the Tor Project umbrella.
