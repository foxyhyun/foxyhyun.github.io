---
title: Foxy Scholar
layout: default
---

# 🦊 Foxy Scholar
논문 리뷰와 연구 노트를 정리하는 개인 블로그입니다.

---

## 📌 Latest Posts

<ul>
  {% for post in site.posts limit:5 %}
    <li>
      <a href="{{ post.url | relative_url }}">{{ post.title }}</a>
      <span>({{ post.date | date: "%Y-%m-%d" }})</span>
    </li>
  {% endfor %}
</ul>
