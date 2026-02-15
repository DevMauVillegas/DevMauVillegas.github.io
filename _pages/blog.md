---
title: "Blog"
permalink: /blog/
layout: single
sidebar:
  nav: "projects"
---

{% assign filtered_posts = site.posts | where_exp: "post", "post.categories contains 'blog'" %}

{% for post in filtered_posts %}
  <h1><a href="{{ post.url }}">{{ post.title }}</a></h1>
  <p>{{ post.description }}</p>
  ---
{% endfor %}