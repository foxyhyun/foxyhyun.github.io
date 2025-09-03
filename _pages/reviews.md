---
title: Paper Review
permalink: /reviews/
layout: post

---

# 🧠 Paper Review

<div class="tag-buttons">
  <a href="#all" class="btn" data-target="all">All</a>
  <a href="#basicai" class="btn" data-target="basicai">BasicAI</a>
  <a href="#biomedicalimaging" class="btn" data-target="biomedicalimaging">Biomedical Imaging</a>
  <a href="#imageenhancement" class="btn" data-target="imageenhancement">Image Enhancement</a>
  <a href="#computervision" class="btn" data-target="computervision">Computer Vision</a>
  <a href="#anomalydetection" class="btn" data-target="anomalydetection">Anomaly Detection</a>
</div>

<section class="pv-section" data-key="basicai">
  <h2>BasicAI</h2>
  <ul class="paper-list">
  {% for post in site.posts %}
    {% if post.categories contains "PaperReview" and post.tags contains "BasicAI" %}
      <li><a class="post-link" href="{{ post.url | relative_url }}">{{ post.title }}</a>
        <time>{{ post.date | date: "%Y-%m-%d" }}</time></li>
    {% endif %}
  {% endfor %}
  </ul>
</section>

<section class="pv-section" data-key="biomedicalimaging">
  <h2>Biomedical Imaging</h2>
  <ul class="paper-list">
  {% for post in site.posts %}
    {% if post.categories contains "PaperReview" and post.tags contains "Biomedical Imaging" %}
      <li><a class="post-link" href="{{ post.url | relative_url }}">{{ post.title }}</a>
        <time>{{ post.date | date: "%Y-%m-%d" }}</time></li>
    {% endif %}
  {% endfor %}
  </ul>
</section>

<section class="pv-section" data-key="imageenhancement">
  <h2>Image Enhancement for Microscopy Image</h2>
  <ul class="paper-list">
  {% for post in site.posts %}
    {% if post.categories contains "PaperReview" and post.tags contains "ImageEnhancement" %}
      <li><a class="post-link" href="{{ post.url | relative_url }}">{{ post.title }}</a>
        <time>{{ post.date | date: "%Y-%m-%d" }}</time></li>
    {% endif %}
  {% endfor %}
  </ul>
</section>

<section class="pv-section" data-key="computervision">
  <h2>Computer Vision</h2>
  <ul class="paper-list">
  {% for post in site.posts %}
    {% if post.categories contains "PaperReview" and post.tags contains "ComputerVision" %}
      <li><a class="post-link" href="{{ post.url | relative_url }}">{{ post.title }}</a>
        <time>{{ post.date | date: "%Y-%m-%d" }}</time></li>
    {% endif %}
  {% endfor %}
  </ul>
</section>

<section class="pv-section" data-key="anomalydetection">
  <h2>Anomaly Detection</h2>
  <ul class="paper-list">
  {% for post in site.posts %}
    {% if post.categories contains "PaperReview" and post.tags contains "AnomalyDetection" %}
      <li><a class="post-link" href="{{ post.url | relative_url }}">{{ post.title }}</a>
        <time>{{ post.date | date: "%Y-%m-%d" }}</time></li>
    {% endif %}
  {% endfor %}
  </ul>
</section>

<script>
(function(){
  const btns = document.querySelectorAll('.btn[data-target]');
  const secs = document.querySelectorAll('.pv-section');
  function show(key){
    secs.forEach(s => s.style.display = (key==='all'||s.dataset.key===key)?'block':'none');
    btns.forEach(b => b.classList.toggle('active', b.dataset.target===key));
    if(key!=='all') location.hash = key;
  }
  btns.forEach(b => b.addEventListener('click', e => { e.preventDefault(); show(b.dataset.target); }));
  window.addEventListener('DOMContentLoaded', () => {
    const key = (location.hash||'#all').slice(1);
    show(key);
  });
})();
</script>
