---
layout: page
title: Blockchain Explorers
---

{::options parse_block_html="true" /}

### With support for name and currency operations

[Cyphrs Namecoin Explorer](https://namecoin.cyphrs.com/) (free software; consensus-safe)<br>
[namecha.in](https://namecha.in/) (non-free software)<br>
[Namebrow.se](https://www.namebrow.se/) (non-free software)<br>

### Currency operations only

{% assign shuffled_explorers_currency = site.data.explorers_currency | sample: 3 %}{% for i in shuffled_explorers_currency %}{{ i }}<br>{% endfor %}

To get on the list please post on the [forum](https://forum.namecoin.org).
