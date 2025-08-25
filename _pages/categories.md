---
layout: default
title: Categories
permalink: /categories/
---

# 📂 Categories

<ul>
  {% for category in site.categories %}
    <li>{{ category[0] }} ({{ category[1].size }} 글)</li>
  {% endfor %}
</ul>
