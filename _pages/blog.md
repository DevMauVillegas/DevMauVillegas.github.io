---
title: "Blog"
permalink: /blog/
layout: single
---

{% assign filtered_posts = site.posts | where_exp: "post", "post.categories contains 'blog'" %}

{% for post in filtered_posts %}
  <h2><a href="{{ post.url }}">{{ post.title }}</a></h2>
  <p>{{ post.description }}</p>
  ---
{% endfor %}