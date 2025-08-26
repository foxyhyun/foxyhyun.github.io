---
layout: post

title: Tags
permalink: /tags/
---

# 🏷️ Tags

<ul>
  {% for tag in site.tags %}
    <li>{{ tag[0] }} ({{ tag[1].size }} 글)</li>
  {% endfor %}
</ul>
