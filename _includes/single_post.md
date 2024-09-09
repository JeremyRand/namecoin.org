<div class="post">

<header class="post-header">
<h1 class="post-title">{% if permalinks %}<a href="{{ post.url | relative_url }}">{% endif %}{{ post.title }}{% if permalinks %}</a>{% endif %}</h1>
<p class="post-meta">{{ post.date | date: "%b %-d, %Y" }}{% if post.author %} • {{ post.author }}{% endif %}{% if post.meta %} • {{ post.meta }}{% endif %} {{tags_content}}</p>
</header>

<article class="post-content">
{{ post.content }}

</article>

<footer>
<p><em>If you liked this article, maybe consider <a href='{{ "/donate/" | relative_url }}'>donating</a>, <a href="https://namecoin.creator-spring.com/">buying a t-shirt</a>, or <a href='{{ "/contribute/" | relative_url }}'>contributing in some other way</a>. Funding from average users like you helps keep us independent. Thanks for your support!</em></p>
</footer>

</div>
