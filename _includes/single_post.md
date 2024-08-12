<div class="post">

<header class="post-header">
<h1 class="post-title">{% if permalinks %}<a href="{{ post.url | relative_url }}">{% endif %}{{ post.title }}{% if permalinks %}</a>{% endif %}</h1>
<p class="post-meta">{{ post.date | date: "%b %-d, %Y" }}{% if post.author %} • {{ post.author }}{% endif %}{% if post.meta %} • {{ post.meta }}{% endif %} {{tags_content}}</p>
</header>

<article class="post-content">
{{ post.content }}

*If you liked this article, maybe consider [donating]({{ "/donate/" | relative_url }}), [buying a t-shirt](https://namecoin.creator-spring.com/), or [contributing in some other way]({{ "/contribute/" | relative_url }}). Funding from average users like you helps keep us independent. Thanks for your support!*
</article>

</div>
