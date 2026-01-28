---
title: "Documentation posts"
permalink: /documentation/
---

{% assign filtered_posts = site.posts | where_exp: "post", "post.categories contains 'documentation'" %}

{% for post in filtered_posts %}
  <h2><a href="{{ post.url }}">{{ post.title }}</a></h2>
  <p>{{ post.description }}</p>
  ---
{% endfor %}
