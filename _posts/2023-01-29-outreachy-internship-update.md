---
layout: post
title: "Outreachy Internship Progress"
author: "Robert Nganga"
tags: [News]
---

Greetings! I wanted to provide an overall update on the project's progress. We have successfully implemented the ability to detect the IP address and port that socket system calls are sending data to, as well as the capability to block system calls that are sending data to an IP address and port that is not the desired proxy. Additionally, we have added an option to kill the application if a proxy leak occurs, which is useful for manual QA testing, and an option to allow proxy leaks but log any that occur, which is useful for automated testing of applications. Furthermore, we have included the capability to use the environment variables that Tor Browser uses, such as TOR_SOCKS_PORT, to determine the desired proxy. 

Lastly, we are currently in the process of implementing SOCKSification, which involves intercepting the connect syscall's entry point, modifying the destination IP/Port, and capturing the established socket's file descriptor through the exit of the connect syscall. A SOCKS5 handshake is then performed on the file descriptor. Afterwards, socks authentication enforcement and support of UDP proxies will be implemented. Socks authentication enforcement will detect if the tracee is using SOCKS without an appropriate username/password (stream isolation leak)

The program instructions are manipulated using flags through [easyconfig](https://github.com/hlandau/easyconfig/tree/v1.0.18) which enables easy manipulation of arguments. Logging is done using [xlog](https://github.com/hlandau/xlog) which provides different options for logging. [U-root](https://github.com/u-root/u-root/tree/v0.10.0/pkg/strace) was preferred for tracing since it is easier to use.

Suggestions are much welcomed! [Project Repository](https://github.com/namecoin/heteronculous-horklump) & [Project Summary]({{ "/2022/11/29/introducing-intern-robert-nganga.html" | relative_url }})

This work was funded by [Outreachy](https://www.outreachy.org/) under the Tor Project umbrella.
