---
title: Paper Review
permalink: /reviews/
layout: default
---

# 🧠 Paper Review

원하는 분야를 선택하세요.

<div class="tag-buttons">
  <a href="#basicai" class="btn">BasicAI</a>
  <a href="#medical" class="btn">Medical</a>
  <a href="#aixmedical" class="btn">AI × Medical</a>
  <a href="#imageenhancement" class="btn">Image Enhancement</a>
  <a href="#computervision" class="btn">Computer Vision</a>
  <a href="#anomalydetection" class="btn">Anomaly Detection</a>
</div>

## BasicAI {#basicai}
<ul class="paper-list">
{% for post in site.posts %}
  {% if post.categories contains "PaperReview" and post.tags contains "BasicAI" %}
    <li><a class="post-link" href="{{ post.url | relative_url }}">{{ post.title }}</a>
      <time>{{ post.date | date: "%Y-%m-%d" }}</time></li>
  {% endif %}
{% endfor %}
</ul>

## Medical {#medical}
<ul class="paper-list">
{% for post in site.posts %}
  {% if post.categories contains "PaperReview" and post.tags contains "Medical" %}
    <li><a class="post-link" href="{{ post.url | relative_url }}">{{ post.title }}</a>
      <time>{{ post.date | date: "%Y-%m-%d" }}</time></li>
  {% endif %}
{% endfor %}
</ul>

## AI × Medical {#aixmedical}
<ul class="paper-list">
{% for post in site.posts %}
  {% if post.categories contains "PaperReview" and post.tags contains "AIxMedical" %}
    <li><a class="post-link" href="{{ post.url | relative_url }}">{{ post.title }}</a>
      <time>{{ post.date | date: "%Y-%m-%d" }}</time></li>
  {% endif %}
{% endfor %}
</ul>

## Image Enhancement {#imageenhancement}
<ul class="paper-list">
{% for post in site.posts %}
  {% if post.categories contains "PaperReview" and post.tags contains "ImageEnhancement" %}
    <li><a class="post-link" href="{{ post.url | relative_url }}">{{ post.title }}</a>
      <time>{{ post.date | date: "%Y-%m-%d" }}</time></li>
  {% endif %}
{% endfor %}
</ul>

## Computer Vision {#computervision}
<ul class="paper-list">
{% for post in site.posts %}
  {% if post.categories contains "PaperReview" and post.tags contains "ComputerVision" %}
    <li><a class="post-link" href="{{ post.url | relative_url }}">{{ post.title }}</a>
      <time>{{ post.date | date: "%Y-%m-%d" }}</time></li>
  {% endif %}
{% endfor %}
</ul>

## Anomaly Detection {#anomalydetection}
<ul class="paper-list">
{% for post in site.posts %}
  {% if post.categories contains "PaperReview" and post.tags contains "AnomalyDetection" %}
    <li><a class="post-link" href="{{ post.url | relative_url }}">{{ post.title }}</a>
      <time>{{ post.date | date: "%Y-%m-%d" }}</time></li>
  {% endif %}
{% endfor %}
</ul>
