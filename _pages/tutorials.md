---
title: "Tutorials"
permalink: /tutorials/
layout: default
---

# 📚 Tutorials

<div class="tag-buttons">
  <a class="btn active" href="#all">All</a>
  <a class="btn" href="#image-processing">Image Processing</a>
  <a class="btn" href="#computer-vision">Computer Vision</a>
  <a class="btn" href="#deep-learning">Deep Learning</a>
  <a class="btn" href="#math">Math / Stats</a>
</div>

<!-- All -->
<section class="pv-section" id="all">
  <h2>All</h2>
  <ul class="paper-list">
  {% for p in site.posts %}
    {% if p.categories contains "Tutorial" %}
      <li>
        <a class="post-link" href="{{ p.url | relative_url }}">{{ p.title }}</a>
        <time>{{ p.date | date: "%Y-%m-%d" }}</time>
      </li>
    {% endif %}
  {% endfor %}
  </ul>
</section>

<!-- Image Processing -->
<section class="pv-section" id="image-processing">
  <h2>Image Processing</h2>
  <ul class="paper-list">
  {% for p in site.posts %}
    {% if p.categories contains "Tutorial" and p.tags contains "ImageProcessing" %}
      <li><a class="post-link" href="{{ p.url | relative_url }}">{{ p.title }}</a>
        <time>{{ p.date | date: "%Y-%m-%d" }}</time></li>
    {% endif %}
  {% endfor %}
  </ul>
</section>

<!-- Computer Vision -->
<section class="pv-section" id="computer-vision">
  <h2>Computer Vision</h2>
  <ul class="paper-list">
  {% for p in site.posts %}
    {% if p.categories contains "Tutorial" and p.tags contains "ComputerVision" %}
      <li><a class="post-link" href="{{ p.url | relative_url }}">{{ p.title }}</a>
        <time>{{ p.date | date: "%Y-%m-%d" }}</time></li>
    {% endif %}
  {% endfor %}
  </ul>
</section>

<!-- Deep Learning -->
<section class="pv-section" id="deep-learning">
  <h2>Deep Learning</h2>
  <ul class="paper-list">
  {% for p in site.posts %}
    {% if p.categories contains "Tutorial" and p.tags contains "DeepLearning" %}
      <li><a class="post-link" href="{{ p.url | relative_url }}">{{ p.title }}</a>
        <time>{{ p.date | date: "%Y-%m-%d" }}</time></li>
    {% endif %}
  {% endfor %}
  </ul>
</section>

<!-- Math / Stats -->
<section class="pv-section" id="math">
  <h2>Math / Stats</h2>
  <ul class="paper-list">
  {% for p in site.posts %}
    {% if p.categories contains "Tutorial" and (p.tags contains "Math" or p.tags contains "Stats") %}
      <li><a class="post-link" href="{{ p.url | relative_url }}">{{ p.title }}</a>
        <time>{{ p.date | date: "%Y-%m-%d" }}</time></li>
    {% endif %}
  {% endfor %}
  </ul>
</section>
