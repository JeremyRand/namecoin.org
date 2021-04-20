---
layout: post
title: Namecoin Response to Recent No-IP Seizure
author: Namecoin Developers
tags: [News]
redirect_from:
  - /post/91894977885/
  - /post/91894977885/namecoin-response-to-recent-no-ip-seizure/
---
On June 30, 2014, customers of the No-IP dynamic DNS service suddenly experienced an outage.  According to media reports, the outage was not caused by a technical issue or any kind of negligence by No-IP, but rather by a court order requested by Microsoft, supposedly to fight cybercrime.  For background, see [No-IP’s official statement](https://www.noip.com/blog/2014/06/30/ips-formal-statement-microsoft-takedown/?utm_source=email&utm_medium=notice&utm_campaign=letter-from-dan).
 
First off -- we only know what was reported by the media.  However, we are very concerned that a US judge decided to shut down an immensely popular service provider due to a few abuses by a tiny minority of its users, particularly with no due process afforded to No-IP.
 
At Namecoin, we believe that decentralization is the best way to prevent malicious takedowns.  The only way to be certain that your domain will not be collateral damage to this kind of abusive court action is to make sure that it is mathematically infeasible to take it down without your consent.  Bad lawyers can’t change the laws of math.
 
Namecoin supports dynamic DNS via the .bit top-level domain and the DyName software by Namecoin developer Jeremy Rand.  DyName has existed since October 2013, and both Jeremy and Namecoin developer Daniel Kraft have invested development effort to improve the security of this use case.  Dynamic DNS was tricky to implement securely with Namecoin, because updates have to be done automatically on a machine with network access, and if the wallet (which must be unencrypted to be used automatically) is compromised, control of the name could permanently be transferred to the attacker.  We circumvented this by using a Namecoin specification feature called the “import” field.  Under this system, the keys necessary to transfer control of your domain, or to impersonate the server it points to, remain on an encrypted wallet.  If the keys used to update your IP are stolen, the worst the attacker can do is make your website go down until you notice and change keys (your visitors will receive a TLS error upon visiting your website in the meantime).  Our software for accessing .bit domains, NMControl (which is used by FreeSpeechMe), supports this feature.
 
Namecoin also supports simulated dynamic DNS via Tor and I2P services.  If you configure a .onion or .b32.i2p domain using Tor or I2P, you can give this domain a human-memorable name using Namecoin.  The Namecoin software FreeSpeechMe has supported this for HTTP websites since December 2013.  While bandwidth and latency are worse than standard IPv4, this method does not require any port forwarding or IPv6 connectivity (DyName requires one of these).
 
The system is not yet perfect.  DyName DNS updates take approximately 10 minutes to propagate the network, and up to 30 minutes after that to be refreshed in the NMControl cache.  The Tor and I2P service support does not yet support HTTPS and non-HTTP connections.  Due to build issues, we haven’t yet been able to release a FreeSpeechMe update which includes the latest NMControl with support for DyName domains.  And the Namecoin-Qt GUI does not yet make it easy to set up DyName or Tor/I2P .bit domains.
 
We believe that all of the above shortcomings are fixable.  However, Namecoin developers are working on this in their free time, and we don’t have a significant financial stake in Namecoin like most altcoin developers do.  If you’d like to support decentralized dynamic DNS to help prevent the next takedown, please consider contributing to a [bounty]({{ site.forum_url }}/viewforum.php?f=18) for the features you want more attention paid to.  And if you’re a developer and you want to help, get in touch with us on the [forum]({{ site.forum_url }}/).
