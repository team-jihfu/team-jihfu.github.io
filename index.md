---
layout: page
title: Posts
---
{% include JB/setup %}

<ul class="posts">
  {% assign sortedposts = site.posts | sort %}
  {% for post in sortedposts %}
  {% assign author = site.data.authors[page.author] %}
    <li>
	<h2>
	    <a href="{{ BASE_PATH }}{{ post.url }}">{{ post.title }}</a>
	</h2>
	<span>Posted by <em>{{ site.data.authors[post.author].name }}</em> on {{ post.date | date_to_string }} in {{ post.category }}</span>
    </li>
  {% endfor %}
</ul>
