---
title: "[Review] Unsupervised Attention-guided Image-to-Image Translation (NeurIPS 2018)"
categories: [PaperReview]
tags: [Computer Vision]
year: "2018"
venue: "NeurIPS 2018"
paper: "https://arxiv.org/abs/1806.02311"
---

<figure>
  <img src="/assets/paper_img/attention_gan/fig1.png" alt="Figure 1" style="max-width:100%; border-radius:12px;">
</figure>

<blockquote><b>왜 이 논문을 선택했는가?</b></blockquote>

<p>기존 <b>CycleGAN</b> 계열 모델은 내가 원하는 부분만 바꾸기보다
<b>이미지 전체가 함께 변형</b>되는 문제가 있었다. 
현재 <b>현미경 영상에서 세포(cell) 영역을 향상(Enhancement)</b>하는 연구를 진행 중인 필자는, 
변환 과정에서 <b>세포 이외의 배경까지 함께 바뀌는 현상</b>을 최소화하고자 했다.</p>

<p>이 논문의 핵심은 간단하다. <mark>어텐션 맵(attention map)</mark>을 학습해 
<b>변환이 필요한 영역만 선택적으로 바꾸고, 배경은 최대한 보존</b>하는 것이다.</p>
<p>그 결과, <b>더 자연스럽고 의미(구조) 보존이 잘 되는</b> 번역 이미지를 얻을 수 있다.</p>
<p>이제부터 본 논문의 아이디어와 방법을 <b>조금 더 자세히</b> 살펴보자.</p>

<figure>
  <img src="/assets/paper_img/attention_gan/fig2.png" alt="Background figure" style="max-width:100%; border-radius:12px;">
</figure>

<p><b>Image-to-Image Translation</b> : 한 도메인의 이미지를 다른 도메인으로 변환하는 것.<br>
핵심은 내용/구조는 유지하면서 스타일/도메인만 변경하는 것이다.</p>

<p>문제는, <b>CycleGAN</b>처럼 전체 이미지를 다루면 배경, 조명, 텍스처까지 바뀌어 구조 보존이 어려울 수 있다는 것이다.</p>
