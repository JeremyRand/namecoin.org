---
layout: post
title: "MoneroKon 5 Summary"
author: Jeremy Rand
tags: [News]
---

As was [previously announced]({{ "/2025/06/15/monerokon-5.html" | relative_url }}), I (Jeremy Rand) filled in for Robert Nganga representing Namecoin at MoneroKon 5 in Prague, Czechia (2025 June 20-22). As usual for conferences that Namecoiners attend, I engaged in a large number of conversations with other attendees.  Also as usual, we won't be publicly disclosing the content of those conversations, because I want people to be able to talk to us at conferences without worrying that off-the-cuff comments will be broadcast to the public.

MoneroKon's official recording is below:

## MoneroKon 5: SocksTrace: A Proxy Leak Detector for Anonymity-Focused Network Applications

Speaker: Robert Nganga (Jeremy Rand filling in)

Monero has excellent transaction-level privacy, and things should get even better soon with FCMP++s. But these innovations aren't sufficient if you have deanonymization vectors on the network level. Tor gives you network privacy in theory — but how do you know your Monero wallet isn't leaking some network traffic outside of Tor? Proxy leaks are a surprisingly common class of security vulnerability, and they've historically been difficult to audit for.

**SocksTrace** is a tool designed to detect proxy leaks by intercepting network syscalls using `seccomp`. SocksTrace is suitable for usage in CI testing as well as manual QA testing, and it can also SOCKSify connections if used with applications that don’t support Tor. This talk will cover the technical design of SocksTrace, how our approach compares to existing tools like Whonix and torsocks, and how Monero application developers can integrate SocksTrace into their workflows.

[Video is here.](https://www.youtube.com/watch?v=l1BkLI9bDlA)

## Thanks

This work was funded by NLnet Foundation's NGI0 Core Fund and Cyphrs.
