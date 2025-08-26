---
layout: default
title: "Home"
---

## 🦊 Foxy Scholar

간단한 논문 리뷰와 연구 노트를 기록하는 공간입니다.  
주 관심사는 **Biomedical Imaging / Image Enhancement / Signal Analysis / Anomaly Detection** 입니다.

<div class="tag-buttons">
  <a class="btn" href="{{ '/about/' | relative_url }}">About 자세히 보기</a>
  <a class="btn" href="{{ '/reviews/' | relative_url }}">🧠 Paper Review</a>
</div>

### Latest
<ul class="paper-list">
{% assign recent = site.posts | where_exp:'p','p.categories contains "PaperReview"' | slice: 0, 5 %}
{% for post in recent %}
  <li>
    <a class="post-link" href="{{ post.url | relative_url }}">{{ post.title }}</a>
    <time>{{ post.date | date: "%Y-%m-%d" }}</time>
  </li>
{% endfor %}
</ul>
