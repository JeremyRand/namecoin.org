---
layout: post
title: "Outreachy Internship Progress"
author: "Robert Nganga"
tags: [News]
---

SOCKSification refers to the process of rerouting network traffic from an application via a SOCKS proxy server. This method is frequently used for security, anonymity, and circumventing network constraints. There are numerous techniques for SOCKSification, but in this article, we will focus on two main approaches that were considered.

The first method includes intercepting the application's connect and send system calls. When an application wants to connect to a remote server, it performs a connect system call with the target IP address and port specified. Following that, the send system call is used to transfer data through the established connection.

With this method, we intercept the connect syscall and change the target IP address and port to those of the SOCKS proxy server. Then we intercept the send syscall which will contain a pointer (in tracee memory space) to the buffer that the application wants to send. We can prepend a SOCKS handshake to that buffer.  While this method can accomplish SOCKSification, it necessitates changing the tracee's memory, which is not memory safe and might cause security problems.

The second method, which is safer, includes utilizing the pidfd getfd capability, which was added to Linux v5.6, to duplicate the file descriptor of the established connection. When an application connects, the connect syscall returns a file descriptor, which may be used to transmit and receive data through the connection. We can replicate the file descriptor from the tracee to the tracer using the pidfd getfd functionality without having to touch the tracee's memory.

To SOCKSify the program, we intercept the connect syscall and change the target IP address and port to those of the SOCKS proxy server. The exit of the connect syscall, which returns the file descriptor of the established socket, is then intercepted. Using pidfd getfd, we copy this file descriptor to the tracer and execute a SOCKS5 handshake with the tracer's socket. After the handshake, we resume the tracee, and all future network traffic gets SOCKSified.

SOCKSification occurs in the function [Socksify.](https://github.com/robertmin1/heteronculous-horklump/blob/test-3/main.go#L412) This function allows an application running on a Linux system to use the SOCKS5 protocol to route network traffic through a proxy server.

The function begins by checking whether the "one circuit" configuration option is set. If not, it initializes a set of authentication data consisting of 10 SOCKS user+pass pairs. Each connection is then assigned to a randomly selected slot from those 10. This is important in order to prevent vulnerability to a Sybil attack. Applications that connect to a P2P network (such as the Bitcoin network) may need to avoid using the same Tor circuit for all connections. Otherwise, a malicious Tor exit relay can block the application's view of the network. Furthermore, using a different Tor circuit for each connection can also be helpful for things like download managers, as it avoids slowdowns caused by an unlucky slow Tor circuit. 

The function then reads the IP address and port number of the address we're diverting from exit_addr, a thread-safe map. The address for each process that requires the SOCKS5 proxy is saved in the map.

The function then opens the file descriptor associated with the program's network connection. With the os.NewFile function, the file descriptor is utilized to construct a new file object. The net.FileConn function is then called on the file object to generate a net.Conn object. This enables the application to communicate with its network connection using the standard net package rather than the lower-level system calls that were used to establish the connection.

The NewClient function is called, as well as the client's Dial method. This creates a new client containing details such as authentication credentials and timeout values. The Dial method is then called, which carries out the redirecting. To listen to an existing net.Conn , we are currently utilizing a customized version of SOCKS5.

### Relevant Links

[SOCKS5 Repository](https://github.com/txthinking/socks5) We are planning to contribute the changes to the main SOCKS5 repository (Allowing the Dial Function to listen to an existing net.Conn file)
