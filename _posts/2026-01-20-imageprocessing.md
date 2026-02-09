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
이미지를 얻는 과정에서 Noise를 완전히 피하기란 거의 불가능합니다.  
즉, 우리가 보는 이미지는 우리가 원하는 이상적인 이미지와 측정 과정에서 발생한 Noise가 더해진 결과라고 볼 수 있죠.  
따라서 이미지는 다음과 같이, y = x + n으로 표현됩니다.  
이 관점에서 보면, denoising은 이 noise 성분을 어떻게 제거할 것인가의 문제입니다.  

<figure>
  <img src="/assets/tutorial_img/0120/fig2.png" alt="Figure 2" style="max-width:100%; border-radius:12px;">
</figure>

<blockquote><b>Type of Noise</b></blockquote>
또한 Noise에는 다양한 유형이 있습니다.  
보통 Bio쪽에서 많이 언급되는 Noise들은 Gaussian Noise, Salt-and-Pepper Noise, Poisson Noise, Speckle Noise가 있습니다.  

<figure>
  <img src="/assets/tutorial_img/0120/fig3.png" alt="Figure 3" style="max-width:100%; border-radius:12px;">
</figure>

<blockquote><b>Gaussian Noise</b></blockquote>
먼저 Gaussian Noise입니다.  
Gaussian Noise는 Noise의 Intensity가 Gaussian Distribution을 따르는 Noise를 의미합니다.  
단순히 덧셈으로 표현되기에 이 더해진 n을 빼는 것이 Gaussian Noise제거의 목표입니다.  
에를들어, thermal fluctuation이나 sensor의 electronic noise가 있습니다.  

<figure>
  <img src="/assets/tutorial_img/0120/fig4.png" alt="Figure 4" style="max-width:100%; border-radius:12px;">
</figure>

<blockquote><b>Salt-and-Pepper Noise</b></blockquote>
Salt-and-Pepper Noise입니다.  
주로 Bit Error나 Faulty Pixel에 의해 발생하고, 각각 숫자를 기록하는 과정에서 0/1이 잘못 써지거나, 아예 고장난 픽셀 하나가 이상한 값을 리턴할 때 생기는 Noise입니다.  
보통 이미지의 밝기에 해당하는 픽셀값이 0부터 255의 값을 가질 때, 이 Noise는 일부 픽셀이 0이나 255로 강제로 바뀌면서 발생합니다.  
픽셀값이 0일 때는 어두워서 Pepper라고 부르고, 255일 때는 밝아서 Salt라고 부릅니다.  

<figure>
  <img src="/assets/tutorial_img/0120/fig5.png" alt="Figure 5" style="max-width:100%; border-radius:12px;">
</figure>

<blockquote><b>Poisson Noise</b></blockquote>
다음은 Poisson Noise입니다.  
이 데이터는 측정 자체에 확률적 변동이 있다고 가정합니다.  
예를들어, intensity가 100인 무언가를 측정한다고 생각해볼게요. 
하지만 실제 측정된 값은 94, 106, 99, 101일 수 있습니다.  
이러한 차이는 Poisson Distribution을 따른다고 가정하며, 이 측정의 차이를 Noise로 취급합니다.  

<figure>
  <img src="/assets/tutorial_img/0120/fig6.png" alt="Figure 6" style="max-width:100%; border-radius:12px;">
</figure>

<blockquote><b>Speckle Noise</b></blockquote>
마지막으로 Speckle Noise입니다.  
Speckle Noise는 이전 Noise들과는 다르게 y = x*n처럼 signal에 곱해지는 형태의 Noise입니다.  
주로 파동 base source인 laser 기반 imaging이나 Ultrasound에서 나타나며, 파동이 서로 겹치면서 생기는 간섭 패턴에 의해 생깁니다.  
재밌게도, Noise를 Multiply하기 때문에 구조가 조금 움직여도 Noise가 같이 따라오는 구조라서 Noise를 추적하여 이동량을 측정하는 연구도 있다고하네요.  

<figure>
  <img src="/assets/tutorial_img/0120/fig7.png" alt="Figure 7" style="max-width:100%; border-radius:12px;">
</figure>
그럼 노이즈를 없애는 방법은 어떤게 있을까요?  
노이즈를 없애는 방법인 Denoising을 설명하기 전에, 유사 개념인 Restoration, Enhancement도 알아보죠.  

먼저, Denoising은 말 그대로 Noise만 제거하는 것이 목표입니다.  
Blur나 시스템 왜곡은 고려하지 않으며 오로지 잡음을 제거하는 방법입니다.  
대표적으로 Mean/Median/Gaussian Filter가 있으며 가장 흔하게 사용되는 방법입니다.  

다음으로 Restoration입니다.  
Restoration은 이미지가 어떤 방식, 즉 어떤 시스템에 의해 망가졌다고 모델링하고, 그 역문제를 풀어 원본을 복원하려는 접근입니다.  

마지막으로 Image Enhancement입니다.  
Enhancement는 원본 복원보다는 사람이 보기에 더 잘보이도록 시각적 품질을 개선하는 데 초점을 둡니다.  
대표적으로 Histogram Equalization, CLAHE등이 있습니다.  

<figure>
  <img src="/assets/tutorial_img/0120/fig8.png" alt="Figure 8" style="max-width:100%; border-radius:12px;">
</figure>
그렇다면 왜 디노이징을 하려는걸까요?  
예를 들어 우리가 이미징을 할 때, 노출 시간을 늘리면 Photobleaching, Phototoxicity가 증가합니다.  
즉, 이미지를 좋게 보려고 노력하더라도 다른 이미지 품질 저하 문제가 생길 수 있다는것이죠.  
따라서 Noise 자체가 없다는 말이 안되며, 좋은 이미지를 얻는다는 것은 높은 SNR을 얻는다는 말과 같습니다.  
  
SNR을 높이는 방법에는 물리적으로 Background를 줄이는 방법이 있습니다.  
배경을 줄이고 샘플에 집중하는 장비, Confoca과 Two-photon Microscopy같은 장비들을 쓰면 도움이 될 수 있죠.  
또한 더 좋은 염색체를 사용하는 방법도 있습니다.  
마지막으로 AI를 사용하여 Noise를 줄이는 방법이 있습니다.  

<figure>
  <img src="/assets/tutorial_img/0120/fig9.png" alt="Figure 9" style="max-width:100%; border-radius:12px;">
</figure>
이제 본격적으로 Image Denoising 방법들에 대해서 알아보겠습니다.  
먼저 대표적이고 고전적인 방법들입니다.  
Mean Filter는 각각의 Pixel을 주변 이웃의 평균으로 대체하는 필터입니다.  
따라서 이미지가 조금 더 Smooth해지며 Noise가 줄어들 수 있죠.  
하지만 Edge또한 Blur가 되거나 detail이 손상될 수 있다는 단점이 있습니다.  
  
Median Filter는 각각의 Pixel을 주변 이웃의 중앙값으로 대체하는 필터입니다.  
중앙값을 사용하기에 평균보다는 이상치에 대해 더 강하죠.  
따라서 일반적인 Denoising에는 Median Filter가 강점이 있습니다.  
하지만 이 방법 또한 detail을 손상시킬 수 있다는 한계가 있죠.  
