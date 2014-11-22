{% capture date %}{{ post.date }}{% endcapture %}
{% capture this_year %}{{ date | date: "%Y" }}{% endcapture %}

<article>
	<h3><a href="{{ root_url }}{{ post.url }}">{{post.title}}</a></h3>
	<div class="meta">
		<span class="date">{{ date | date: "%b %e" }}</span>
	</div>
</article>
