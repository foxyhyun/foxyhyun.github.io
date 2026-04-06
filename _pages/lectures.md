---
title: "Lectures"
permalink: /lectures/
layout: default
---

# Lectures

<p class="lectures-subtitle">주제별로 정리된 강의 노트입니다. 순서대로 따라가면 개념을 체계적으로 익힐 수 있습니다.</p>

<div class="course-grid">

  <!-- Computer Vision -->
  {% assign cv_lectures = site.lectures | where: "course", "computer-vision" | sort: "order" %}
  {% if cv_lectures.size > 0 %}
  <div class="course-card">
    <div class="course-card-header cv">
      <span class="course-icon">👁️</span>
      <h2 class="course-card-title">Computer Vision</h2>
    </div>
    <p class="course-card-desc">이미지와 영상을 컴퓨터가 이해하는 방법을 다룹니다. 픽셀 표현부터 딥러닝 기반 인식 모델까지 단계적으로 학습합니다.</p>
    <div class="course-meta">
      <span class="course-count">{{ cv_lectures.size }} Lectures</span>
    </div>
    <ol class="course-preview-list">
      {% for lec in cv_lectures limit: 3 %}
        <li><a href="{{ lec.url | relative_url }}">{{ lec.title }}</a></li>
      {% endfor %}
      {% if cv_lectures.size > 3 %}
        <li class="more">+ {{ cv_lectures.size | minus: 3 }} more</li>
      {% endif %}
    </ol>
    <a class="course-start-btn" href="{{ cv_lectures.first.url | relative_url }}">Start Course →</a>
  </div>
  {% endif %}

  <!-- Basic of AI -->
  {% assign ai_lectures = site.lectures | where: "course", "basic-ai" | sort: "order" %}
  {% if ai_lectures.size > 0 %}
  <div class="course-card">
    <div class="course-card-header ai">
      <span class="course-icon">🤖</span>
      <h2 class="course-card-title">Basic of AI</h2>
    </div>
    <p class="course-card-desc">AI와 머신러닝의 기초 개념을 쉽게 설명합니다. 수학적 배경부터 주요 알고리즘의 원리까지 차근차근 다룹니다.</p>
    <div class="course-meta">
      <span class="course-count">{{ ai_lectures.size }} Lectures</span>
    </div>
    <ol class="course-preview-list">
      {% for lec in ai_lectures limit: 3 %}
        <li><a href="{{ lec.url | relative_url }}">{{ lec.title }}</a></li>
      {% endfor %}
      {% if ai_lectures.size > 3 %}
        <li class="more">+ {{ ai_lectures.size | minus: 3 }} more</li>
      {% endif %}
    </ol>
    <a class="course-start-btn" href="{{ ai_lectures.first.url | relative_url }}">Start Course →</a>
  </div>
  {% endif %}

</div>
