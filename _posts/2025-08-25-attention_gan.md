---
title: "[Computer Vision] Unsupervised Attention-guided Image-to-Image Translation (NeurIPS 2018)"
categories: [PaperReview]
tags: [ComputerVision]
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

<blockquote><b>Background</b></blockquote>

<figure>
  <img src="/assets/paper_img/attention_gan/fig2.png" alt="Background figure" style="max-width:100%; border-radius:12px;">
</figure>

<p><b>Image-to-Image Translation</b> : 한 도메인의 이미지를 다른 도메인으로 변환하는 것.<br>
핵심은 내용/구조는 유지하면서 스타일/도메인만 변경하는 것이다.</p>

<p>문제는, <b>CycleGAN</b>처럼 전체 이미지를 다루면 배경, 조명, 텍스처까지 바뀌어 구조 보존이 어려울 수 있다는 것이다.</p>

<figure>
  <img src="/assets/paper_img/attention_gan/fig3.png" alt="CycleGAN limitation" style="max-width:100%; border-radius:12px;">
</figure>

<p>이전 모델(CycleGAN)의 한계는 전체를 바꾼다는 것.<br>
즉, 우리가 원하는 관심 부분(ROI)만 변경하는 것이 아닌, 불필요한 <b>background</b>도 함께 변환하는 것이다.</p>

<p>따라서 이 논문의 핵심 아이디어는 다음과 같다.</p>

<p style="text-align:center;"><mark><b>“Translation Network가 어디를 바꿔야 하는지(foreground),<br>
어디는 유지해야 하는지(background)를 스스로 Attention Map으로 학습하게 만들자”</b></mark></p>

<figure>
  <img src="/assets/paper_img/attention_gan/fig4.png" alt="Attention idea" style="max-width:100%; border-radius:12px;">
</figure>

<blockquote><b>Method - Overview</b></blockquote>

<figure>
  <img src="/assets/paper_img/attention_gan/fig5.png" alt="Method overview" style="max-width:100%; border-radius:12px;">
</figure>

<p>전체 구조는 다음과 같습니다. <b>CycleGAN</b>과 마찬가지로 양방향 생성기로, 
 \(F_{S \rightarrow T}, F_{T \leftarrow S}\)  로 구성된다. <br>
<b>Attention Network</b>는 입력마다  \(A_S(s), A_T(t)\)  라는 <b>soft attention map</b>을 0~1 사이로 예측한다. <br>
<b>Discriminator</b>는  \(D_T, D_S\)  로 구성되며, 주의 영역만 보도록 입력을 마스킹한다.</p>

<p>이 때, 최종 <b>output</b>은 <b>foreground</b>와 <b>background</b>를 합친 값이다.</p>

<blockquote>Attention-guided Generator</blockquote>

<figure>
  <img src="/assets/paper_img/attention_gan/fig6.png" alt="Attention-guided Generator" style="max-width:100%; border-radius:12px;">
</figure>

<p><b>Generator</b>는  \(A_S(s)\) 라는 0과 1 사이 값으로 구성된 가중치 맵을 배운다. 
이 가중치 맵을 보고 변환할지, 그대로 둘지 결정한다. <br>
예를 들어, \(A_S = 1\)  이면 변환을 채택,  \(A_S = 0\)  이면 원본을 유지한다.</p>

<p><b>Loss Function</b>은 <b>Adversarial Loss</b>와 <b>Cycle Consistency Loss</b>로 구성된다. 
<mark><b>Adversarial Loss</b>는 진짜/가짜를 구별하고, <b>Cycle Consistency Loss</b>는 원본과 생성된 이미지의 차이를 줄인다.</mark><br>
Cycle Consistency의 가중치(람다)가 커지면 내용 보존이 강해지고, 작아지면 내용 변화가 커질 위험이 있다. (논문에서는 \lamda =10 사용)</p>

<p><b>Adversarial Loss</b>에서 <i>real target</i> 이미지는 1로, 생성 이미지는 0으로 판별하도록 학습한다.<br>
<b>Cycle Consistency Loss</b>는 L1로 원본과 생성물의 차이를 줄여 더 진짜 같은 이미지를 만들도록 유도한다.</p>

<p>두 손실을 합친 <b>최종 목적함수</b>는 GAN의 <b>minimax</b>로 풀며, 
생성기/어텐션은 <b>minimize</b>, 판별기는 <b>maximize</b>한다.</p>

<blockquote>Attention-guided Discriminator</blockquote>

<figure>
  <img src="/assets/paper_img/attention_gan/fig7.png" alt="Attention-guided Discriminator" style="max-width:100%; border-radius:12px;">
</figure>

<p><b>Discriminator</b>가 전체 이미지를 보면 배경 통계를 학습해 <b>Generator</b>가 배경까지 변환하는 현상이 생길 수 있다. 
이를 막기 위해 <b>Attention</b> 값이 높은 영역만 보도록 입력을 마스킹한다. <br>
먼저 <b>threshold</b> 값을 사용하여 입력 마스크를 만들고(논문에선 0.1), 
마스킹된 영역에만 <b>Adversarial Loss</b>를 계산한다.</p>

<p><b>학습 주의사항</b>: 초기 attention map은 랜덤에 가깝기 때문에, 
처음부터 마스킹하면 틀린 영역을 강화할 수 있다. 따라서 초기(약 0–30 epoch)에는 전체 이미지를 사용해 
판별기를 안정화시키며 attention이 형성되도록 한다. 이후 <mark>epoch 30 이후</mark>에는 
threshold로 이진화한 attention mask를 적용한다.</p>

<blockquote>Results</blockquote>

<figure>
  <img src="/assets/paper_img/attention_gan/fig8.png" alt="Results" style="max-width:100%; border-radius:12px;">
</figure>

<p>결과적으로 <b>CycleGAN</b> 등 기존 모델보다, 우리가 원하는 <b>ROI</b> 부분만 선택적으로 변경되는 모습을 확인할 수 있다.</p>