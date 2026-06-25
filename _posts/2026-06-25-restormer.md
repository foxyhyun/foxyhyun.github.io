---
title: "[Image Restoration] Restormer: Efficient Transformer for High-Resolution Image Restoration"
categories: [PaperReview]
tags: [ComputerVision, Restoration, Transformer, Denoising, Deblurring, Deraining]
year: "2022"
venue: "CVPR 2022"
layout: post
paper: "https://arxiv.org/abs/2111.09881"
---

## Motivation

<blockquote><b>왜 Image Restoration 에 Transformer 인가?</b></blockquote>

<p>
<b>Image Restoration</b> 은 열화된(degraded) 이미지에서 깨끗한 이미지를 복원하는 <mark>ill-posed</mark> 문제다.
해가 무수히 많기 때문에, "이미지가 원래 어떻게 생겼어야 하는지"에 대한 사전 지식, 즉 <b>image prior</b> 가 필요하다.
</p>

<p>
기존의 <b>CNN</b> 기반 방법은 데이터로부터 이 prior 를 효과적으로 학습한다.
CNN 의 강점은 <b>local connectivity</b>(각 뉴런이 인접 픽셀만 본다)와 <b>translation equivariance</b>(위치에 무관하게 패턴 인식)에서 온다.
그러나 두 가지 근본적 한계가 있다.
</p>

<ul>
  <li><b>제한된 receptive field</b> : 멀리 떨어진 픽셀 정보(long-range dependency)를 보려면 레이어를 깊게 쌓아야 한다.</li>
  <li><b>static weights</b> : convolution filter 는 inference 시 가중치가 고정되어 있어, 입력 내용에 유연하게 적응하지 못한다.</li>
</ul>

<p>
<b>Self-Attention(SA)</b> 은 이 두 한계를 모두 해결한다.
전체 이미지의 모든 픽셀 쌍 간 유사도를 계산하므로 <b>receptive field 가 전체 이미지</b> 이고, 입력에 따라 동적으로 가중치가 결정된다.
하지만 SA 의 연산량은 공간 해상도에 대해 <b>제곱(quadratic)으로 증가</b> 하기 때문에,
고해상도 이미지를 다루는 대부분의 image restoration 작업에는 적용하기 어렵다.
</p>

<blockquote>
We propose an efficient Transformer for image restoration that is capable of modeling global connectivity and is still applicable to large images.
</blockquote>

<p>
Restormer 는 Transformer 의 핵심 building block(multi-head attention, feed-forward network)을 재설계하여,
<b>long-range pixel interaction 을 포착하면서도 고해상도 이미지에 적용 가능한 선형 복잡도(linear complexity)</b> 를 달성한다.
</p>

## Overall Pipeline

<figure>
  <img src="/assets/paper_img/restormer/fig1.png" alt="Restormer architecture" style="max-width:100%; border-radius:12px;">
</figure>

<p>
열화 이미지 \( I \in \mathbb{R}^{H \times W \times 3} \) 가 주어지면, 전체 흐름은 다음과 같다.
</p>

<ul>
  <li>먼저 convolution 으로 low-level feature embedding \( F_0 \in \mathbb{R}^{H \times W \times C} \) 를 얻는다.</li>
  <li>이 shallow feature 는 <b>4-level 대칭 encoder-decoder</b> 를 거쳐 deep feature \( F_d \in \mathbb{R}^{H \times W \times 2C} \) 로 변환된다.</li>
  <li>각 level 은 여러 개의 <b>Transformer block</b> 으로 구성되며, 효율을 위해 <b>아래 level 로 갈수록 block 수를 점진적으로 늘린다</b>.</li>
  <li>해상도 조절에는 정보 손실을 막기 위해 <b>pixel-unshuffle</b>(해상도↓·채널↑)과 <b>pixel-shuffle</b> 을 사용한다 (max pooling/interpolation 대신).</li>
  <li>마지막으로 refine 된 feature 에 convolution 을 적용해 <b>residual image</b> \( R \) 을 만들고, 입력을 더해 복원 이미지를 얻는다 : \( \hat{I} = I + R \).</li>
</ul>

<p>
특히 <b>level-1</b> 에서는 Transformer block 이 encoder 의 low-level feature 와 decoder 의 high-level feature 를 함께 집계(aggregate)하여,
미세한 구조 디테일을 보존하도록 설계되었다.
</p>

<p>
Transformer block 은 두 개의 핵심 모듈로 이루어진다 — <b>MDTA</b>(attention 부분)와 <b>GDFN</b>(feed-forward 부분).
</p>

## ① MDTA : Multi-Dconv Head Transposed Attention

<blockquote><b>공간이 아니라 채널 방향으로 attention 을 적용한다</b></blockquote>

<p>
vanilla multi-head SA 가 공간 차원에서 픽셀 쌍 간 attention 을 계산해 제곱 복잡도를 갖는 것과 달리,
MDTA 는 <b>채널(feature) 차원에서 attention 을 적용</b> 한다.
즉, 픽셀 쌍 상호작용을 명시적으로 모델링하는 대신, key·query 로 투영된 입력 feature 의
<b>cross-covariance(채널 간 공분산)</b> 를 계산해 attention map 을 만든다.
이렇게 하면 복잡도가 공간 해상도에 대해 <mark>선형(linear)</mark> 이 되어, 고해상도 이미지에도 적용 가능하다.
</p>

<p>
MDTA 의 또 다른 핵심은 공분산 계산 <b>이전</b> 에 수행하는 <b>local context mixing</b> 이다.
</p>

<ul>
  <li><b>1×1 convolution</b> : 채널 간 context 를 픽셀 단위로 집계(pixel-wise aggregation of cross-channel context).</li>
  <li><b>depth-wise convolution</b> : 각 채널 내에서 주변 픽셀의 local context 를 집계(channel-wise aggregation of local context).</li>
</ul>

<p>
이 전략은 두 가지 이점을 준다. 첫째, 공간적으로 인접한 <b>local context</b> 를 강조하여 convolution 의 강점을 파이프라인에 결합한다.
둘째, covariance 기반 attention map 을 계산하는 과정에서 픽셀 간 <b>전역적 관계(global relationship)가 암묵적으로(implicitly) 모델링</b> 되도록 보장한다.
</p>

## ② GDFN : Gated-Dconv Feed-Forward Network

<blockquote><b>Gating 으로 정보 흐름을 제어한다</b></blockquote>

<p>
기존 feed-forward network 의 첫 번째 linear transformation 을 <b>gating mechanism</b> 으로 재구성하여 네트워크의 정보 흐름을 개선한다.
gating 은 두 갈래의 선형 경로 중 하나에 <b>GELU</b> 비선형성을 적용한 뒤 element-wise 곱으로 결합하는 방식이다.
GELU 값이 0 에 가까우면 해당 위치의 feature 를 억제하고, 1 에 가까우면 통과시킨다.
</p>

<p>
GDFN 은 각 계층(hierarchical level)을 거치는 정보 흐름을 제어하여,
<b>각 level 이 다른 level 과 상보적인(complementary) 미세 디테일에 집중</b> 하도록 만든다.
여기에도 depth-wise convolution 이 들어가 local 구조를 인코딩한다.
</p>

<p>
한편, 네트워크 전반에 <b>bias-free convolutional layer</b> 를 사용한다.
bias 항이 있으면 다양한 noise level 에 대해 성능이 떨어지는데,
bias 를 제거하면 입력의 <b>절대값이 아니라 상대적 패턴</b> 에 집중할 수 있어 일반화에 유리하다.
</p>

## Progressive Learning

<p>
학습과 테스트의 이미지 해상도가 다르면 inference 성능이 떨어질 수 있다.
이를 완화하기 위해 Restormer 는 <b>progressive learning</b> 을 사용한다 —
학습 초반에는 작은 patch 로 학습하고, 후반 epoch 으로 갈수록 점점 더 큰 patch 로 학습한다.
이는 쉬운 것부터 어려운 것 순으로 학습하는 <mark>curriculum learning</mark> 의 한 형태로,
학습 때 본 적 없는 크기에도 모델이 대응할 수 있게 한다.
(각 image restoration task 마다 <b>별도의 모델</b> 을 학습한다.)
</p>

## Results

<figure>
  <img src="/assets/paper_img/restormer/fig2.png" alt="Image deraining results" style="max-width:100%; border-radius:12px;">
</figure>

<p>
<b>Image Deraining</b> 에서 Restormer 는 여러 벤치마크에 걸쳐 기존 방법 대비 일관된 PSNR/SSIM 향상을 보인다(평균 PSNR 최대 +1.05 dB).
실제 비 내리는(real rainy) 이미지에서도 더 선명하고 충실한(faithful) 결과를 만든다.
</p>

<figure>
  <img src="/assets/paper_img/restormer/fig3.png" alt="Single image motion deblurring on GoPro" style="max-width:100%; border-radius:12px;">
</figure>

<p>
<b>Single Image Motion Deblurring</b>(GoPro)에서도 Restormer 는 번호판 텍스트처럼 복원이 어려운 영역까지
가장 선명하고 시각적으로 충실한 결과를 만든다(26.96 dB, 비교군 중 최고).
</p>

## Conclusion

<p>
Restormer 는 MDTA 와 GDFN 이라는 두 핵심 설계로 SA 의 제곱 복잡도 문제를 우회하여,
<b>전역 연결성(global connectivity)을 모델링하면서도 고해상도 이미지에 적용 가능한 효율적 Transformer</b> 를 제안했다.
deraining, deblurring, denoising 등 다양한 image restoration task 에서 SOTA 를 달성하며,
Transformer 가 복원 문제의 강력한 백본이 될 수 있음을 보였다.
</p>
