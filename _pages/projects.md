---
title: "Projects"
permalink: /projects/
---

{% assign projects = site.pages %}

{% for post in projects %}
  <h2><a href="{{ post.url }}">{{ post.title }}</a></h2>
  <p>{{ post.description }}</p>
  ---
{% endfor %}
