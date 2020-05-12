---
layout: page
title: Exchanges
---

{::options parse_block_html="true" /}

<span style="font-size:130%;">&#8644;</span> You can buy and sell **Namecoins** here:

<span id="decentralized-exchanges" style="font-size:130%;">
**Decentralized exchanges**<br>
{% assign shuffled_exchanges_decentralized = site.data.exchanges_decentralized | sample: 2 %}{% for i in shuffled_exchanges_decentralized %}{{ i }}<br>{% endfor %}
</span>

<span style="font-size:130%;">
**Gold level exchanges**<br>
...<br>
</span>

<span style="font-size:115%;">
**Silver level exchanges**<br>
...<br>
</span>

<span style="font-size:100%;">
**Bronze level exchanges**<br>
...<br>
</span>

<span style="font-size:85%;">
**Basic level exchanges**<br>
{% assign shuffled_exchanges_basic = site.data.exchanges_basic | sample: 10 %}{% for i in shuffled_exchanges_basic %}{{ i }}<br>{% endfor %}
</span>

Gold, silver and bronze level exchanges donate to the Namecoin project. The higher the donation the higher the level. Note that the exchanges are not verified in any way.
The current fees are 1000/300/100NMC per year. To get on the list contact exchanges a|t namecoin.org
