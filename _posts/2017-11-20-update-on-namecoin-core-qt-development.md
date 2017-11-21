---
layout: post
title: "Update on Namecoin Core Qt Development"
author: Brandon Roberts
tags: [News]
---

It's been roughly a year since the [initial manage names tab](https://github.com/namecoin/namecoin-core/pull/67)
code was ported from legacy Namecoin to Namecoin Core. Since then, development
of namecoin-qt has been progressing on two fronts: merging the manage names Qt
interface into Namecoin Core's master branch and the development of a `d/` spec
DNS configuration interface.

In October, I initiated a [pull request](https://github.com/namecoin/namecoin-core/pull/187)
to merge the manage names tab into Namecoin Core's master branch.
This patch replaces the previous, experimental manage names (v0.13.99) interface and
brings it current with Namecoin Core version 0.15.99. There have been many
bugfixes and improvements, so we welcome users to test the code and report any
issues.

Secondly, thanks to [support](https://www.namecoin.org/2017/05/19/funding-nlnet.html)
from the NLnet Foundation's Internet Hardening Fund, I've began developing the
DNS configuration dialog. The goal is to make managing Namecoin domains much simpler
and will remove the need for many users to build `d/` spec JSON documents
altogether. We currently have a set of [mocks available](https://github.com/namecoin/namecoin-core/issues/196)
for comments from users. Until a pull request is issued, development can be tracked
in the [manage-dns](https://github.com/brandonrobertz/namecoin-core/tree/manage-dns)
branch of my Namecoin Core repo.

Development is moving quickly, so I will continue to update the community as
things progress. As usual, you can follow our work in the [GitHub repo](https://github.com/namecoin/namecoin-core)
and on the [Namecoin Subreddit](https://www.reddit.com/r/Namecoin/).

