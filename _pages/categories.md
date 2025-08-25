---
layout: default
title: Categories
permalink: /categories/
---

# 📂 Categories

여기서는 제가 사용한 **카테고리별 글 모음**을 확인할 수 있습니다.  
(예: PaperReview, Author 등)

<ul>
  {% for category in site.categories %}
    <li>
      {{ category[0] }} ({{ category[1].size }} 글)
    </li>
  {% endfor %}
</ul>
