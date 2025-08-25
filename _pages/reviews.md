---
title: Paper Review
permalink: /reviews/
layout: default
---

# 🧠 Paper Review

아래 버튼을 눌러 분야별 리뷰 목록을 볼 수 있어요.  
(글에 붙이는 **태그**는 정확히: `BasicAI`, `Medical`, `AIxMedical`, `ImageEnhancement`, `ComputerVision`, `AnomalyDetection`)

<div class="tag-buttons">
  <a href="#basicai" class="btn">BasicAI</a>
  <a href="#medical" class="btn">Medical</a>
  <a href="#aixmedical" class="btn">AI × Medical</a>
  <a href="#imageenhancement" class="btn">Image Enhancement</a>
  <a href="#computervision" class="btn">Computer Vision</a>
  <a href="#anomalydetection" class="btn">Anomaly Detection</a>
</div>

---

## 🔹 BasicAI {#basicai}
<ul>
{% for post in site.posts %}
  {% if post.categories contains "PaperReview" and post.tags contains "BasicAI" %}
    <li><a href="{{ post.url | relative_url }}">{{ post.title }}</a> <span>({{ post.date | date: "%Y-%m-%d" }})</span></li>
  {% endif %}
{% endfor %}
</ul>

## 🔹 Medical {#medical}
<ul>
{% for post in site.posts %}
  {% if post.categories contains "PaperReview" and post.tags contains "Medical" %}
    <li><a href="{{ post.url | relative_url }}">{{ post.title }}</a> <span>({{ post.date | date: "%Y-%m-%d" }})</span></li>
  {% endif %}
{% endfor %}
</ul>

## 🔹 AI × Medical {#aixmedical}
<ul>
{% for post in site.posts %}
  {% if post.categories contains "PaperReview" and post.tags contains "AIxMedical" %}
    <li><a href="{{ post.url | relative_url }}">{{ post.title }}</a> <span>({{ post.date | date: "%Y-%m-%d" }})</span></li>
  {% endif %}
{% endfor %}
</ul>

## 🔹 Image Enhancement {#imageenhancement}
<ul>
{% for post in site.posts %}
  {% if post.categories contains "PaperReview" and post.tags contains "ImageEnhancement" %}
    <li><a href="{{ post.url | relative_url }}">{{ post.title }}</a> <span>({{ post.date | date: "%Y-%m-%d" }})</span></li>
  {% endif %}
{% endfor %}
</ul>

## 🔹 Computer Vision {#computervision}
<ul>
{% for post in site.posts %}
  {% if post.categories contains "PaperReview" and post.tags contains "ComputerVision" %}
    <li><a href="{{ post.url | relative_url }}">{{ post.title }}</a> <span>({{ post.date | date: "%Y-%m-%d" }})</span></li>
  {% endif %}
{% endfor %}
</ul>

## 🔹 Anomaly Detection {#anomalydetection}
<ul>
{% for post in site.posts %}
  {% if post.categories contains "PaperReview" and post.tags contains "AnomalyDetection" %}
    <li><a href="{{ post.url | relative_url }}">{{ post.title }}</a> <span>({{ post.date | date: "%Y-%m-%d" }})</span></li>
  {% endif %}
{% endfor %}
</ul>
