---
title: "[Image Enhancement for Microscopy Image] Correction of out-of-focus microscopic images by deep learning"
categories: [PaperReview]
tags: [ImageEnhancement]
year: "2022"
venue: "Computational and Structural Biotechnology Journal"
layout: post
paper: "https://www.sciencedirect.com/science/article/pii/S2001037022001192"
---

## Introduction
<figure>
  <img src="C:/Users/sihyun/Desktop/foxyhyun.github.io/assets/paper_img/oofgan/fig1.png" alt="Introduction figure" style="max-width:100%; border-radius:12px;">
</figure>

<blockquote><b>Motivation</b></blockquote>

<p>
현미경으로 얻은 <b>Microscopy images</b> 는 촬영 환경이나 초점 세팅의 한계로 인해 
자주 <mark>out-of-focus</mark> 상태로 기록된다. 
이렇게 초점이 맞지 않은 이미지는 세포 구조나 조직 경계를 흐릿하게 만들어,
연구 및 진단에서 <b>성능 저하</b>를 일으킨다.
</p>

<p>
특히 임상·연구 환경에서는 동일 샘플을 다시 촬영하기 어려운 경우가 많기 때문에, 
이미 촬영된 <mark>out-of-focus Microscopy images</mark> 를 
<b>in-focus</b> 수준으로 복원하는 <b>Deblurring</b> 알고리즘의 필요성이 크다.
</p>

<blockquote><b>Method</b></blockquote>

<p>
이 논문은 이미지 도메인 변환에 널리 사용되는 <b>CycleGAN</b> 구조를 기반으로,
<mark>out-of-focus Microscopy images</mark> 를 
<b>in-focus</b> 스타일의 이미지로 변환하는 <b>Deblurring 네트워크</b>를 제안한다.
</p>

<p>
이를 위해 단일 손실이 아니라 여러 손실 항으로 구성된 
<mark>multi-component weighted loss function</mark> 을 도입하여, 
단순히 이미지를 선명하게 만드는 것을 넘어 
<b>구조 보존</b>과 <b>텍스처 유지</b>, 
그리고 <b>Ground Truth in-focus 이미지</b> 와의 전반적인 시각적 일관성을 
동시에 향상시키는 것을 목표로 한다.
</p>
