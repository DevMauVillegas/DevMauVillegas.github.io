---
title: "Blog"
permalink: /blog/
layout: single
author_profile: false
---

{% assign filtered_posts = site.posts | where_exp: "post", "post.categories contains 'blog'" %}

{% for post in filtered_posts %}
  <h3><a href="{{ post.url }}">{{ post.title }}</a></h3>
  <p>{{ post.excerpt }}</p>
{% endfor %}