---
layout: default
---
{% include post_tags.html %}

{% for post in site.posts %}

{% if post.id == page.id %}

{% include single_post.md %}

{% endif %}

{% endfor %}