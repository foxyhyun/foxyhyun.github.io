---
title: "[Denoising] Introduction of Image Denoising & Enhancement for Microscopic Image"
categories: [Tutorial]
tags: [ImageProcessing]
year: "2026"
layout: post
---

<figure>
  <img src="/assets/tutorial_img/0120/fig1.png" alt="Figure 1" style="max-width:100%; border-radius:12px;">
</figure>

<blockquote><b>What is the Noise?</b></blockquote>
이미지를 얻는 과정에서 Noise를 완전히 피하기란 거의 불가능합니다. \
즉, 우리가 보는 이미지는 우리가 원하는 이상적인 이미지와 측정 과정에서 발생한 Noise가 더해진 결과라고 볼 수 있죠. \
따라서 이미지는 다음과 같이, y = x + n으로 표현됩니다. \
이 관점에서 보면, denoising은 이 noise 성분을 어떻게 제거할 것인가의 문제입니다. \

<figure>
  <img src="/assets/tutorial_img/0120/fig2.png" alt="Figure 2" style="max-width:100%; border-radius:12px;">
</figure>

<blockquote><b>Type of Noise</b></blockquote>
또한 Noise에는 다양한 유형이 있습니다. \
보통 Bio쪽에서 많이 언급되는 Noise들은 Gaussian Noise, Salt-and-Pepper Noise, Poisson Noise, Speckle Noise가 있습니다. \

<figure>
  <img src="/assets/tutorial_img/0120/fig3.png" alt="Figure 3" style="max-width:100%; border-radius:12px;">
</figure>

<blockquote><b>Gaussian Noise</b></blockquote>
먼저 Gaussian Noise입니다.
Gaussian Noise는 Noise의 Intensity가 Gaussian Distribution을 따르는 Noise를 의미합니다. \
단순히 덧셈으로 표현되기에 이 더해진 n을 빼는 것이 Gaussian Noise제거의 목표입니다. \
에를들어, thermal fluctuation이나 sensor의 electronic noise가 있습니다. \

<figure>
  <img src="/assets/tutorial_img/0120/fig4.png" alt="Figure 4" style="max-width:100%; border-radius:12px;">
</figure>

<blockquote><b>Salt-and-Pepper Noise</b></blockquote>
Salt-and-Pepper Noise입니다. \
주로 Bit Error나 Faulty Pixel에 의해 발생하고, 각각 숫자를 기록하는 과정에서 0/1이 잘못 써지거나, 아예 고장난 픽셀 하나가 이상한 값을 리턴할 때 생기는 Noise입니다. \
보통 이미지의 밝기에 해당하는 픽셀값이 0부터 255의 값을 가질 때, 이 Noise는 일부 픽셀이 0이나 255로 강제로 바뀌면서 발생합니다. \
픽셀값이 0일 때는 어두워서 Pepper라고 부르고, 255일 때는 밝아서 Salt라고 부릅니다. \

<figure>
  <img src="/assets/tutorial_img/0120/fig5.png" alt="Figure 5" style="max-width:100%; border-radius:12px;">
</figure>

<blockquote><b>Poisson Noise</b></blockquote>
다음은 Poisson Noise입니다.  \
이 데이터는 측정 자체에 확률적 변동이 있다고 가정합니다. \
예를들어, intensity가 100인 무언가를 측정한다고 생각해볼게요. \
하지만 실제 측정된 값은 94, 106, 99, 101일 수 있습니다. \
이러한 차이는 Poisson Distribution을 따른다고 가정하며, 이 측정의 차이를 Noise로 취급합니다. \

<figure>
  <img src="/assets/tutorial_img/0120/fig6.png" alt="Figure 6" style="max-width:100%; border-radius:12px;">
</figure>

<blockquote><b>Speckle Noise</b></blockquote>
마지막으로 Speckle Noise입니다. \
Speckle Noise는 이전 Noise들과는 다르게 y = x*n처럼 signal에 곱해지는 형태의 Noise입니다. \
주로 파동 base source인 laser 기반 imaging이나 Ultrasound에서 나타나며, 파동이 서로 겹치면서 생기는 간섭 패턴에 의해 생깁니다. \
재밌게도, Noise를 Multiply하기 때문에 구조가 조금 움직여도 Noise가 같이 따라오는 구조라서 Noise를 추적하여 이동량을 측정하는 연구도 있다고하네요. \

