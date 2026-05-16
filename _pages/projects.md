---
title: "Projects"
permalink: /projects/
sidebar:
  nav: "projects"
---

# Select the project you want to check out


{% for post in site.pages %}
{% if post.url contains '/projects/' and post.image %}
## <a href="{{ post.title }}">
- ### Project type: {{post.type }}
- ### {{ post.description }}
<a href="{{ post.url }}">
<img src="{{ post.image }}" alt="{{ post.title }}">
</a>

---

{% endif %}
{% endfor %}
