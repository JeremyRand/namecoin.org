---
layout: post
title: "Go DNS Seeder TCP Support"
author: Jeremy Rand
tags: [News]
---

In a previous post, I covered some Go DNS seeder improvements I made.  Now here's another one: DNS over TCP support.

DNS is an application-layer protocol, and can thus be used with multiple transport protocols.  The most common is UDP, but various others such as TCP, TLS, DTLS, QUIC, HTTP, HTTPS, and Unix sockets are a thing.  TCP is particularly relevant from a privacy standpoint because TCP works with Tor, while UDP does not.  Due to a bug, Lyndsay Roger's DNS seeder didn't support TCP, which made it unpleasant to access over Tor.  I've now fixed that; master branch now supports DNS over TCP, which both improves the experience for Tor users and brings the DNS seeder into compliance with RFC 7766 (which says TCP support is mandatory).

Thanks to Lyndsay for merging the fix!  The fix will hopefully be deployed onto Namecoin's DNS seeds over the next days.
