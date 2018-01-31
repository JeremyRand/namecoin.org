---
layout: post
title: "Recent Reports of Ransomware Using Namecoin are Missing the Real Story"
author: Jeremy Rand
tags: [News]
---

Some reports are making the rounds that a new ransomware strain, "GandCrab", is using Namecoin for C&C.  While this may sound interesting, as far as I can tell these reports are missing the real story.

Looking at the [report on Bleeping Computer](https://www.bleepingcomputer.com/news/security/gandcrab-ransomware-distributed-by-exploit-kits-appends-gdcb-extension/), we see these quotes:

> Another interesting feature is GandCrab's use of the NameCoin .BIT top-level domain.  .BIT is not a TLD that is recognized by the Internet Corporation for Assigned Names and Numbers (ICANN), but is instead managed by [NameCoin's](https://namecoin.org/) decentralized domain name system.

> The developers of GandCrab are using NameCoin's DNS as it makes it harder for law enforcement to track down the owner of the domain and to take the domains down.

This doesn't make much sense, since Namecoin isn't anonymous (so tracking down the owner of the domain is relatively straightforward for law enforcement).  But more to the point, most Internet users don't have Namecoin installed, and it would be rather odd for ransomware to bundle a Namecoin name lookup client.  This confusion is explained by Bleeping Computer (to their credit):

> This means that any software that wishes to resolve a domain name that uses the .BIT tld, must use a DNS server that supports it. GandCrab does this by making dns queries using the a.dnspod.com DNS server, which is accessible on the Internet and can also  be used to resolve .bit domains.

Yep, if this is to be believed, GandCrab isn't actually using Namecoin, they're using a centralized DNS server (`a.dnspod.com`) which nominally claims to mirror the namespace of Namecoin.  This means that, if law enforcement wants to censor the C&C `.bit` domains, they don't need to censor Namecoin (which would be rather difficult), they simply need to look up who owns `dnspod.com` (under ICANN policy, looking this up is straightforward for law enforcement) and send them a court order to censor the C&C domains.

However, Bleeping Computer is actually substantially wrong on this point.  Why?  Take a look in the Namecha.in block explorer at [the Namecoin value of `bleepingcomputer.bit`](https://namecha.in/name/d/bleepingcomputer), which is one of the alleged C&C domains:

~~~
{"ns":["A.DNSPOD.COM","B.DNSPOD.COM"]}
~~~

First off, note that this is actually a completely invalid Namecoin configuration, because [the trailing period is missing](https://help.directadmin.com/item.php?id=541) from the authoritative nameserver addresses, so any DNS software that tries to process that Namecoin domain will return `SERVFAIL`.  Second, note that the authoritative nameservers listed are... the `dnspod.com` nameservers.  This makes it pretty clear that `a.dnspod.com` isn't actually a Namecoin DNS inproxy.  If it were, and even if the trailing-period fail were corrected in the Namecoin value, the inproxy would end up in a recursion loop.  `a.dnspod.com` is actually just a random authoritative nameserver that happens to be serving records for a domain name that ends in `.bit`.  **Namecoin isn't used anywhere by GandCrab, and killing the Namecoin domain wouldn't have any effect on GandCrab.**  Of course, this raises questions about why exactly that domain name is even registered in Namecoin.  The simplest explanations are:

1. The GandCrab developers are massively incompetent, and have potentially deanonymized themselves by registering a Namecoin domain despite not ever using that Namecoin domain for their ransomware.
2. Someone unrelated to GandCrab has registered that Namecoin domain for the purpose of trolling security researchers.

It's conceivable that `dnspod.com`'s nameservers will only allow a `.bit` domain's records to be served from their systems if that `.bit` domain's Namecoin data points to `dnspod.com`'s namservers, and it's further possible that their systems are misconfigured to not notice that the trailing period is missing.  However, this seemed rather unlikely to me.  Why?  Well, first, take a look at the WHOIS data for `dnspod.com`:

~~~
Registrar URL: http://www.dnspod.cn
Registrar: DNSPod, Inc.
Registrar IANA ID: 1697
~~~

So DNSPod is apparently an ICANN-accredited DNS registrar, with a primary domain name in China.  Which of these scenarios fits better with Occam's Razor:

1. A DNS registrar located in China (which is not exactly known for its government's respect for Namecoin's values of free speech), which is accredited by ICANN (which doesn't recognize Namecoin as either a DNS TLD or a special-use TLD), is doing special processing of domain names that they host in order to respect the authority of Namecoin.  They've also never contacted the Namecoin developers to inform us that they're doing this, nor are any of their technical people active in the Namecoin community.
2. DNSPod simply doesn't care what domains their customers host on their nameservers, since if DNSPod's nameservers aren't authorized by a domain's NS record in the DNS, nothing bad will happen anyway (DNSPod simply won't receive any requests for that domain).

In addition, if DNSPod had such a policy, it's not clear how exactly their customers would be able to switch their Namecoin domains to DNSPod nameservers without encountering downtime while DNSPod was waiting for the Namecoin transaction to propagate.

However, since empiricism is informative, Ryan Castellucci tested this with an account on DNSPod, and confirmed that no such validation occurs:

~~~
$ dig +tcp jeremyrandissomesortofhumanperson.bit @a.dnspod.com

; <<>> DiG 9.9.5-9+deb8u14-Debian <<>> +tcp jeremyrandissomesortofhumanperson.bit @a.dnspod.com
;; global options: +cmd
;; Got answer:
;; ->>HEADER<<- opcode: QUERY, status: NOERROR, id: 23586
;; flags: qr aa rd; QUERY: 1, ANSWER: 1, AUTHORITY: 3, ADDITIONAL: 1
;; WARNING: recursion requested but not available

;; OPT PSEUDOSECTION:
; EDNS: version: 0, flags:; udp: 4096
;; QUESTION SECTION:
;jeremyrandissomesortofhumanperson.bit. IN A

;; ANSWER SECTION:
jeremyrandissomesortofhumanperson.bit. 600 IN A 255.255.255.255

;; AUTHORITY SECTION:
jeremyrandissomesortofhumanperson.bit. 600 IN NS b.dnspod.com.
jeremyrandissomesortofhumanperson.bit. 600 IN NS c.dnspod.com.
jeremyrandissomesortofhumanperson.bit. 600 IN NS a.dnspod.com.

;; Query time: 595 msec
;; SERVER: 101.226.79.205#53(101.226.79.205)
;; WHEN: Wed Jan 31 03:04:24 UTC 2018
;; MSG SIZE  rcvd: 160
~~~

(Ryan doesn't own `jeremyrandissomesortofhumanperson.bit`.)  Ryan also did the same for `bleepingcomputer.iq`, implying that DNSPod isn't verifying ownership of DNS domain names either:

~~~
$ dig +tcp bleepingcomputer.iq @a.dnspod.com A

; <<>> DiG 9.9.5-9+deb8u14-Debian <<>> +tcp bleepingcomputer.iq @a.dnspod.com A
;; global options: +cmd
;; Got answer:
;; ->>HEADER<<- opcode: QUERY, status: NOERROR, id: 50149
;; flags: qr aa rd; QUERY: 1, ANSWER: 1, AUTHORITY: 3, ADDITIONAL: 1
;; WARNING: recursion requested but not available

;; OPT PSEUDOSECTION:
; EDNS: version: 0, flags:; udp: 4096
;; QUESTION SECTION:
;bleepingcomputer.iq.           IN      A

;; ANSWER SECTION:
bleepingcomputer.iq.    600     IN      A       8.8.8.8

;; AUTHORITY SECTION:
bleepingcomputer.iq.    600     IN      NS      a.dnspod.com.
bleepingcomputer.iq.    600     IN      NS      b.dnspod.com.
bleepingcomputer.iq.    600     IN      NS      c.dnspod.com.

;; Query time: 5758 msec
;; SERVER: 101.226.79.205#53(101.226.79.205)
;; WHEN: Wed Jan 31 03:00:39 UTC 2018
;; MSG SIZE  rcvd: 142
~~~

Ryan tried registering a `.onion` domain and `bleepingcomputer.malware` on DNSPod as well, but these were rejected as invalid TLD's.  Ryan and I have no clue why `.bit` is on DNSPod's TLD whitelist while `.onion` isn't -- probably because a customer asked for it and DNSPod just doesn't care.

Ryan isn't aware of any prior cases where a malware C&C was set up in a random free authoritative DNS provider such as DNSPod, with the DNS servers hardcoded in the malware.  It's an interesting strategy for malware authors, since authoritative DNS providers usually don't bother to confirm domain name ownership.  Entertainingly, Ryan found that DNSPod isn't verifying ownership of the email addresses used to register accounts either.

So in conclusion: while this is a rather interesting case of a possible hilarious opsec fail by a ransomware author (which very well might get them arrested), and the strategy of using authoritative DNS hosting providers for malware C&C is fascinating as well, the ransomware itself is fully irrelevant to Namecoin.
