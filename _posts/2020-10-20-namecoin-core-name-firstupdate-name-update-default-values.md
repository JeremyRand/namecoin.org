---
layout: post
title: "Namecoin Core name_firstupdate, name_update default values"
author: yanmaani
tags: [News]
---

Namecoin Core will, starting version 0.21, no longer require a value to be provided for the `name_update` and `name_firstupdate` RPC calls.
If no value is provided, Namecoin Core will use the last known such. If none exists, the empty string will be used.
This change has no adverse impact on existing workflows, since it only makes previously required parameters optional.

This change is being made as part of the effort to simplify the RPC API for name management.
