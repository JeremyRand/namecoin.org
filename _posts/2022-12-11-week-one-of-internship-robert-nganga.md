---
layout: post
title: "Internship Progress"
author: "Robert Nganga"
tags: [News]
---

Greetings! I completed the initial tasks for the internship during the first week, where I utilized and experimented with the [u-root strace package.](https://pkg.go.dev/github.com/u-root/u-root/pkg/strace) The first tasks were to launch a program using ptrace and detect socket system calls. The first task was straightforward, requiring only the usage of the [Trace Function.](https://github.com/u-root/u-root/blob/v0.10.0/pkg/strace/tracer.go#L156) For the latter, it included utilizing a map to determine if the system call is a socket syscall and then informing the user if it is.

Later, I worked on the main tasks, which included determining the IP address and port on which socket system calls send data. I did this by first filtering out syscalls that initiate a socket connection, then sending the syscall through the [SysCallEnter function.](https://pkg.go.dev/github.com/u-root/u-root/pkg/strace#SysCallEnter) This returns a string containing the Address and Port, which we then extract using string slicing. Better methods will be developed to replace string slicing.

While resolving some of the [issues](https://github.com/namecoin/heteronculous-horklump/issues), I am now working on blocking system calls that transfer data to an IP address+port that is not the targeted proxy.

This work was funded by Outreachy under the Tor Project umbrella.