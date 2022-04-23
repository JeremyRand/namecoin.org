---
layout: post
title: "Preventing Expiration Mishaps with Semi-Expiration"
author: Jeremy Rand
tags: [News]
---

Forgetting to renew a Namecoin name on time is rather catastrophic: it means that anyone else can re-register it and then hold the name hostage.  In practice today, it is likely that such re-registrations will be done by Good Samaritan volunteers who are happy to donate the name back to you.  However, as per the cypherpunk philosophy of "don't trust, verify", it's not desirable to rely on those Good Samaritans, since they constitute a trusted third party.  How can we improve this situation?

Let's start by observing two fundamental properties of expired names:

1. Expired names resolve as `NXDOMAIN`.
2. Expired names can be registered by anyone.

Astute readers will have noticed that these are, in principle, distinct properties.  Where are they enforced?

1. **Policy layer**: Expired names resolve as `NXDOMAIN`.
2. **Consensus layer**: Expired names can be registered by anyone.

And what are their respective impacts?

1. **Temporary DoS**: Expired names resolve as `NXDOMAIN`.
2. **Permanent hijacking**: Expired names can be registered by anyone.

So what happens if we make the following change?  We introduce a new state for names: *Semi-Expired*.  Semi-expired names obey Property 1 but not Property 2.  Names semi-expire when they come within 4032 blocks (4 weeks) of expiration.  Does the situation improve?

Let's imagine that Alice forgets to renew her name before it semi-expires.  As a result, her name becomes unresolvable.  This causes her website to go down.  Some of Alice's users notice this, and complain to her via email, social media, or some other out-of-band medium.  Alice then renews her name, which reverts the semi-expired state.  And this out-of-band process gives Alice 4 weeks to fix things before she is at risk of having her name stolen.  That certainly seems like an improvement.  And this is solely a policy change, not a consensus change, making it cheap to implement and deploy.

I've now implemented this in Electrum-NMC.  The RPC interface adds fields for the "semi-expired" state, and the Manage Names GUI counts down to semi-expiration rather than expiration.

Some additional observations:

* Semi-expiration will only help you as a name owner if you actually actively use your name.  It won't help you if you hold an unused name for squatting purposes.  While I don't think any of the Namecoin developers are opposed to additional anti-expiry mechanisms that work for squatters, I think it's arguably a good thing that semi-expiration incentivizes active use of names.
* Because semi-expiration decreases the effective duration of resolvability for names (this is a necessary consequence of implementing semi-expiration on the policy layer), it means that name owners will renew somewhat more often with respect to time.  Theoretically, this means that name owners will pay higher fees with respect to time, and that the blockchain will grow faster with respect to time.  In practice, the difference is likely to be negligible, and if it is detectable at all, it will be in the form of increased block reward (which indirectly improves blockchain hashrate) and decreased incentive to squat on names (which indirectly improves usefulness of the system).

Semi-expiration should be released in Electrum-NMC v4.0.0b1.  Namecoin Core will hopefully follow soon after.
