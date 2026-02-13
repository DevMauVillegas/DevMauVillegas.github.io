---
title: "Projects"
permalink: /projects/
---

{% assign projects = site.pages  | where_exp: "page", "page.path contains '_pages/projects/'" %}

{% for post in projects %}
  <a href="{{ post.url }}">
    <img src="{{ post.image }}" alt="{{ post.title }}">
  </a>
{% endfor %}
