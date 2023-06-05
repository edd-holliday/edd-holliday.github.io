---
layout: page
title: Projects
---

Here are some of my projects:

{% for project in site.data.projects %}
- [{{ project.name }}]({{ project.url }}): {{ project.description }}
{% endfor %}
