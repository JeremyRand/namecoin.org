---
layout: post
title: Why Duplicate Namespaces Are Bad for Users
author: Namecoin Developers
tags: [News]
---
Every now and then, we’re approached by users who are asking about namespaces which duplicate functionality of the officially recommended namespaces.  Examples of these namespaces include `tor/`, `i2p/`, and `u/`.  We think these duplicate namespaces are harmful to end users.  This blog post will try to explain why duplicating functionality of existing namespaces is almost always a bad idea.

## Background

This whole thing seems to have started when someone noticed that the `d/` specification, which is used for domain names, only supported IP addresses.  This person thought it would be awesome to have DNS for Tor hidden services, and proposed a `tor/` namespace for such domains.  Someone else appears to have decided the same thing for I2P, and created an `i2p/` proposal.

An alternative proposal was the `d/` 2.0 spec, which allowed `d/` names to map to Tor hidden services and I2P eepsites.  This is the proposal which is for the most part still widely used today.

Why was `d/` 2.0 a better idea than starting 2 new namespaces?

## Duplicate Namespaces Attract Squatters

Imagine that `d/`, `tor/`, and `i2p/` are all common namespaces.  Let’s say that WikiLeaks is running an IPv4-based website using the name `d/wikileaks`.  Now let’s say that WikiLeaks wishes to offer a Tor hidden service for their website.  They now have to register a second name, `tor/wikileaks`.  Uh oh, seems a squatter saw that WikiLeaks had `d/wikileaks` and preemptively grabbed `tor/wikileaks` and `i2p/wikileaks` just in case WikiLeaks wanted those names.  Now WikiLeaks has to negotiate with a squatter, who potentially might be on the payroll of a government who wants to either cost WikiLeaks lots of money or impersonate it.

What’s the defense here?  The only defense is to preemptively register your name in **all** namespaces, just in case you want them later.

Let’s say that in an alternate reality, WikiLeaks did exactly this, registering `d/wikileaks`, `tor/wikileaks`, and `i2p/wikileaks`.  Now let’s imagine that someone decides that CJDNS mesh networking would be nice to have with Namecoin’s domain names, and creates a cjdns/ namespace.  Well crap, now WikiLeaks is in a race with squatters to see who can grab cjdns/wikileaks first; if WikiLeaks fails, then they can’t ever use their name with CJDNS.

Remember that the domain squatters probably expend much more time watching namespace proposals, since that’s basically their job.  WikiLeaks, on the other hand, probably is more worried about their whistleblower anonymity infrastructure than their choice of Namecoin namespaces.

This is simply a losing game for everyone except squatters.

## Duplicate Namespaces Break Users’ Security Assumptions

Imagine a situation where an end user hears from a trusted source that wikileaks.bit is a great place to submit documentation of government corruption.  wikileaks.bit uses the `d/` namespace.  But the end user doesn’t want to use clearnet, so she uses the `tor/` namespace instead, accessing the domain wikileaks.tor.  Note that she has just made the assumption that `d/wikileaks` and `tor/wikileaks` are operated by the same entity.  However, this is not the case.  The Namecoin network considers those two names to be entirely different things.  Imagine that `tor/wikileaks` is instead operated by a government agency (perhaps they bought it from a squatter).  Now our whistleblower is screwed.

If duplicate namespaces are in use, the **only** way to verify that two names are controlled by the same entity is to inspect the name you trust, and see if it links to the other name.  For example, you could visit the website associated with `d/wikileaks`, and see if it mentions `tor/wikileaks`.  This, of course, is neither automatable nor secure in our whistleblower’s case.  Given how popular and successful phishing is, providing scammers with an easy way to break users’ security assumptions is an extremely bad enginering idea.

## Consolidate All Namespaces with Common Use Cases

Let’s look instead at the `d/` 2.0 proposal.  It recognized that IP-based DNS, Tor-based DNS, and I2P-based DNS are all aiming at a common use case.  By creating new functionality as new fields of the `d/` namespace rather than new namespaces, the proposal makes sure that users of Namecoin DNS will be able to freely switch between IP, Tor, I2P, and any future system which may come along, without ever needing to register a new name.  This protects users from squatters and protects their security, both now and in the future.  (It also saves them money on name fees.)

## Your Options When a Namespace Spec Doesn’t Meet Your Needs

Let’s say that you want to implement a new feature into an existing use case in Namecoin.  For example, maybe you wish the `id/` spec had some extra fields.  You have three options:

1. Talk to the other Namecoin developers and propose a change to the spec. This is what we generally recommend, although we realize that in some cases this may be difficult if your project is confidential prior to launch.
2. Add a custom field to an existing namespace spec.  As long as your new field has a name which isn’t likely to collide with other use cases, this is pretty much harmless (although we’d love it if you talked to us).  Even if you’re refactoring existing features, it is likely that software following the existing spec will ignore your extra fields, and you can ignore the preexisting spec’s fields.  This means that the two schemes can co-exist pretty much peacefully.
3. Make a new namespace for your specific fields, and disregard the existing namespace.  **This is harmful; there is almost never a good reason to do this.**

## The `u/` Namespace: Exactly How Not to Approach the Problem

As far as we can tell, a for-profit company (whom we won’t name here) decided to start offering identity products using the Namecoin blockchain.  They didn’t like the structure of the `id/` spec.  So, did they talk to us?  Nope, we never heard of their product until after it launched (we first noticed their product because of a bunch of nonstandard names showing up in the blockchain).  Did they add some custom fields to `id/`?  Nope, they instead did the worst possible thing as listed above: they made their own namespace (`u/`) with identical use cases as the pre-existing `id/`.

## The Squatting Argument Revisited

While we’ve had difficulty reaching the team behind `u/` for an official statement, we are under the impression that one of their stated rationales for creating a new namespace was that `id/` was allegedly squatted too much.  This is a quite strange claim.  Creating a new namespace every time an existing namespace is heavily squatted will disrupt legitimate users of the existing namespace much more than it will disrupt squatters, even if a lot of the existing names are squatted.  It also is an inherently cyclic process, since squatters can grab a new namespace just as fast as the old namespace.  **The correct way to disincentivise squatting (to the extent that it is a problem) is with changes to Namecoin’s fee structure.**

## A “More Structured” Spec

Another commonly cited “advantage” of `u/` is that its JSON structure is allegedly better organized than that of `id/`.  **This is not an adequate reason to start a new namespace.**  Even if the `u/` team didn’t want to directly work with us, they could have simply used their incompatible spec with the `id/` namespace, and everything would have worked just fine.  End users who wanted to use both specs would have been subject to the slight annoyance of having to enter duplicate fields in their existing names, but this would be infinitely better than needing a new name.

## “Decentralization”

Several people have commented that because Namecoin is decentralized, or because it’s a general-purpose naming system, that it’s completely acceptable to create new namespaces without regard for the consequences.  **There is a fundamental difference between centralization and interoperability.**  Centralization would mean that an authority is dictating which names can and cannot be created; this would be extremely bad for end users.  Interoperability means that the developer community is voluntarily standardizing on practices that allow everything to work together with minimal disruption; this directly benefits end users.  We are for interoperability, not centralization.  The ability to create new namespaces is primarily available for creating new applications.  Of course, the blockchain validation math doesn’t care if two namespaces are used for similar things.  This doesn’t mean it’s a good idea, or that people should be doing it on a regular basis.

## Where From Here?

The `u/` team has gotten themselves into a potentially bad situation, since now if they wish to move to `id/`, they’ll have to race against squatters: precisely the reason why we don’t recommend changing namespaces in the first place.  We have limited sympathy for this predicament, since this could have easily been averted had they not created `u/` in the first place.  This has resulted in unnecessary trouble for end users, and we hope that a good resolution can be found.
