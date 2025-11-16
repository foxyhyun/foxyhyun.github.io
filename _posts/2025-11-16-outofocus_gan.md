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
  <img src="/assets/paper_img/oofgan/fig1.png" alt="Introduction figure" style="max-width:100%; border-radius:12px;">
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


<figure>
  <img src="/assets/paper_img/oofgan/fig2.png" alt="Network Architecture" style="max-width:100%; border-radius:12px;">
</figure>

<blockquote><b>Method – Network Architecture</b></blockquote>

<p>
이 논문은 두 도메인 사이의 <b>Image-to-Image Translation</b> 을 수행하는 
<b>CycleGAN</b> 구조를 기반으로 한다.  
여기서 <b>Source 도메인</b>은 <mark>out-of-focus Microscopy image</mark> 이고,  
<b>Target 도메인</b>은 <mark>in-focus Microscopy image</mark> 이다.
</p>

<p>
이를 위해 두 개의 Generator, <b>G<sub>s</sub> : Source → Target</b> 와 
<b>G<sub>t</sub> : Target → Source</b> 를 사용한다.  
각 Generator는 <b>Encoder – Residual Blocks – Decoder</b> 구조로 되어 있어,  
Encoder에서 이미지를 feature 공간으로 압축하고, Residual Blocks에서 
복잡한 패턴과 컨텍스트를 학습한 뒤, Decoder에서 다시 이미지 공간으로 복원한다.
</p>

<p>
각 도메인에는 별도의 Discriminator <b>D<sub>s</sub>, D<sub>t</sub></b> 를 두어  
생성된 이미지가 해당 도메인의 <b>real image</b> 처럼 보이는지 판단한다.  
즉, <b>G</b> 는 <b>D</b> 를 속이도록 이미지를 생성하고,  
<b>D</b> 는 real / fake 를 구분하도록 학습되는 전형적인 <mark>Adversarial learning</mark> 구조이다.
</p>

<hr/>

<figure>
  <img src="/assets/paper_img/oofgan/fig3.png" alt="Multi Component Weighted Loss" style="max-width:100%; border-radius:12px;">
</figure>

<blockquote><b>Method – Multi-Component Weighted Loss</b></blockquote>

<p>
전체 학습은 하나의 손실이 아니라, 세 가지 손실을 합친 
<b>multi-component weighted loss function</b> 으로 이루어진다:
</p>

<p>
1) <b>L<sub>GAN</sub></b> :  
각 도메인 Discriminator(<b>D<sub>s</sub>, D<sub>t</sub></b>) 를 이용해  
생성 이미지가 <b>real in-focus / out-of-focus image</b> 처럼 보이도록 만드는 
<mark>Adversarial loss</mark> 이다.  
쉽게 말하면, <b>“진짜처럼 보이게 만드는 손실”</b> 이다.
</p>

<p>
2) <b>L<sub>cont</sub></b> (content loss):  
사전학습된 네트워크의 <b>feature space</b> 에서  
생성 이미지와 Ground Truth 이미지의 특징을 비교하여,  
<b>구조(structure)와 텍스처(texture)</b> 를 보존하도록 유도하는 손실이다.  
논문에서는 <b>G<sub>s</sub>, G<sub>t</sub></b> 양 방향에 대해 
content loss 를 계산하고 더해,  
세포 모양과 세부 구조가 무너지지 않도록 제어한다.
</p>

<p>
3) <b>L<sub>cycle</sub></b> (cycle-consistency loss):  
out-of-focus 이미지를 <b>G<sub>s</sub></b> 로 in-focus 로 보냈다가,  
다시 <b>G<sub>t</sub></b> 로 원래 도메인으로 되돌렸을 때,  
입력 이미지와 크게 달라지지 않도록 <b>pixel-wise 차이</b>를 줄이는 손실이다.  
이는 <mark>Cycle consistency</mark> 를 보장하여  
비현실적인 변환이나 모드 붕괴를 방지한다.
</p>

<p>
최종 학습 목표는 세 손실을 가중합한
<b>L = λ<sub>1</sub>L<sub>GAN</sub> + λ<sub>2</sub>L<sub>cont</sub> + λ<sub>3</sub>L<sub>cycle</sub></b> 이며,  
각 λ 값은 각각 <b>“얼마나 진짜처럼 보일지”</b>,  
<b>“얼마나 구조·텍스처를 보존할지”</b>,  
<b>“얼마나 변환 전·후를 일관되게 유지할지”</b> 의 비중을 조절하는 하이퍼파라미터로 볼 수 있다.
</p>

<figure>
  <img src="/assets/paper_img/oofgan/fig4.png" alt="Evaluation metrics" style="max-width:100%; border-radius:12px;">
</figure>

<blockquote><b>Evaluation</b></blockquote>

<p>
이 논문에서는 제안한 <b>Deblurring CycleGAN</b> 의 성능을 정량적으로 평가하기 위해  
세 가지 대표적인 화질 지표를 사용한다:  
<b>Peak Signal-to-Noise Ratio (PSNR)</b>,  
<b>Structural Similarity Index (SSIM)</b>,  
<b>Pearson Correlation Coefficient (PCC)</b>.
</p>

<p>
각 지표는 <b>복원된 in-focus 이미지</b>가 <b>Ground Truth in-focus 이미지</b>와  
얼마나 비슷한지를 서로 다른 관점에서 측정한다.  
모든 지표에서 값이 클수록 “더 좋은 복원 결과”를 의미한다.
</p>

<blockquote><b>Peak Signal-to-Noise Ratio (PSNR)</b></blockquote>

<p>
<b>PSNR</b> 은 이미지의 <b>전반적인 노이즈 수준</b>을 보는 지표로,  
원본 신호 에너지에 비해 복원 오차(MSE)가 얼마나 작은지를 로그 스케일로 표현한다.  
값이 클수록 <b>노이즈가 적고, 픽셀 단위 오차가 작다</b>는 뜻이다.
</p>

<blockquote><b>Structural Similarity Index (SSIM)</b></blockquote>

<p>
<b>SSIM</b> 은 단순한 픽셀 차이보다,  
<b>구조(structure), 에지(edge), 콘트라스트(contrast)</b> 가  
얼마나 잘 보존되었는지를 측정하는 지표이다.  
범위는 보통 <b>0 ~ 1</b>이고, 1에 가까울수록  
<b>원본과 구조적으로 매우 유사한 이미지</b>라고 해석할 수 있다.
</p>

<blockquote><b>Pearson Correlation Coefficient (PCC)</b></blockquote>

<p>
<b>PCC</b> 는 두 이미지의 <b>픽셀 intensity 분포</b>가  
얼마나 비슷하게 움직이는지를 보는 <b>상관계수</b>이다.  
범위는 <b>-1 ~ 1</b>이며, 1에 가까울수록  
<b>“밝을 때 같이 밝고, 어두울 때 같이 어두운” 패턴</b>을 잘 유지한다는 의미이다.
</p>

<p>
실험 결과, 제안한 방법은 기존 <b>Pix2Pix, CycleGAN, DeblurGAN</b> 계열보다  
<b>PSNR, SSIM, PCC</b> 모두에서 더 높은 값을 기록하여,  
노이즈 억제뿐 아니라 구조·텍스처 보존과 전체적인 패턴 일치 측면에서도  
우수한 <b>in-focus 복원 성능</b>을 보여준다.
</p>

<figure>
  <img src="/assets/paper_img/oofgan/fig5.png" alt="Qualitative comparison with other methods" style="max-width:100%; border-radius:12px;">
</figure>

<blockquote><b>Result – Comparison with Existing Methods</b></blockquote>

<p>
첫 번째 결과 그림은 <mark>out-of-focus 이미지</mark>를 입력으로 했을 때,  
<b>L0-regularized, Pix2Pix, CycleGAN, DeblurGAN, DeblurGAN-V2</b> 그리고 <b>제안 방법(Ours)</b> 이  
어떤 출력을 내는지 시각적으로 비교한 예시이다.
</p>

<p>
파란색/빨간색 박스로 표시된 <b>국소 영역</b>을 보면,  
기존 방법들은 세포 모양이 흐려지거나, 경계가 과도하게 날카로워지는 등  
<b>노이즈 또는 구조 왜곡</b>이 남아 있는 반면,  
<mark>Ours</mark>는 <b>in-focus 이미지</b>와 가까운 형태로  
세포의 윤곽과 내부 텍스처를 더 잘 복원함을 확인할 수 있다.
</p>

<hr/>

<figure>
  <img src="/assets/paper_img/oofgan/fig6.png" alt="Single FOV correction" style="max-width:100%; border-radius:12px;">
</figure>

<blockquote><b>Result – Out-of-focus → Corrected → In-focus</b></blockquote>

<p>
두 번째 그림은 서로 다른 샘플(A, B, C)에 대해  
<b>Out-of-focus</b>, <b>Corrected(제안 모델 출력)</b>, <b>In-focus</b> 를  
한 줄에서 비교한 결과이다.
</p>

<p>
각 샘플에서 <mark>Corrected</mark> 이미지는  
원래 <b>out-of-focus</b> 이미지보다 명확한 에지와 세포 구조를 보여주며,  
밝기와 콘트라스트도 <b>Ground Truth in-focus</b> 에 가깝게 맞춰져 있다.  
특히 세포 내부의 세밀한 패턴이나 필라멘트 구조가 다시 드러나는 것을 통해  
제안 모델이 단순 샤프닝이 아니라 <b>구조 복원</b>을 수행한다는 점을 알 수 있다.
</p>

<hr/>

<figure>
  <img src="/assets/paper_img/oofgan/fig7.png" alt="Z-stack deblurring across depths" style="max-width:100%; border-radius:12px;">
</figure>

<blockquote><b>Result – Z-stack Across Multiple Depths</b></blockquote>

<p>
세 번째 그림은 서로 다른 <b>Z-depth(Z=5~30)</b> 에서 획득한  
<mark>Out-of-focus_A/B</mark>, 제안 모델의 <mark>Corrected_A/B</mark>,  
그리고 <mark>In-focus_A/B</mark> 를 함께 보여주는 <b>Z-stack 결과</b>이다.
</p>

<p>
Z가 깊어질수록 원본 <b>out-of-focus</b> 이미지는 점점 더 흐려지고  
세포 경계가 사라지지만, 제안된 <b>Corrected</b> 결과는  
각 Z에서 <b>in-focus 이미지와 유사한 구조·패턴</b>을 유지하고 있다.  
이는 본 모델이 단일 깊이에 국한되지 않고,  
<mark>여러 초점 깊이에서 일관된 deblurring 성능</mark>을 낸다는 것을 보여준다.
</p>

<p>
정리하면, 시각적 결과와 정량적 지표(PSNR, SSIM, PCC)를 종합했을 때  
제안한 <b>Deblurring CycleGAN with multi-component weighted loss</b> 는  
기존 방법들보다 <b>구조 보존, 노이즈 억제, 시각적 일관성</b> 측면에서  
더 우수한 out-of-focus 보정 성능을 달성한다.
</p>
