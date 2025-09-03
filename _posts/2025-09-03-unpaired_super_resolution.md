---
title: "[Image Enhancement for Microscopy Image] Unpaired data training enables super-resolution confocal microscopy from low-resolution acquisitions"
categories: [PaperReview]
tags: [ImageEnhancement]
year: "2024"
venue: "Optic Letters"
layout: post
paper: "https://doi.org/10.1364/OL.537713"
---

<figure>
  <img src="/assets/paper_img/0903/fig1.png" alt="Figure 1" style="max-width:100%; border-radius:12px;">
</figure>

<blockquote><b>왜 이 논문을 선택했는가?</b></blockquote>

<p>
<b>LR(Low Resolution), HR(High Resolution)</b> 데이터가 확보된 상태에서 <mark>꼭 Supervised Learning으로만 해야하는가?</mark> 하는 
의문이 생겨 기존에 연구하던 Unsupervised라고 할 수 있는 <b>CycleGAN</b> 적용 사례를 살펴보기 위해 찾아보게 되었다.
</p>

<p>
이 논문의 핵심은 간단하다. 
- Paired Dataset 없이 <b>LR과 HR 이미지 집합만으로 학습해서 Confocal Image(공초점 이미지)를 Super Resolution(SR)로 복원한다.</b>
- CycleGAN 방식으로 구조를 잘 보존하면서도 디테일을 살린다.
</p>

<blockquote><b>Background</b></blockquote>
<p>
현미경 영상은 생물학/의학 연구에서 핵심적인 역할을 하지만, 고해상도 영상을 얻기 위해서는 긴 노출 시간과 강한 광원이 필요합니다.  
이는 시료 손상, 실험 효율 저하 등 문제를 유발합니다.  
따라서 연구자들은 저해상도(LR) 영상으로부터 고해상도(HR) 영상을 복원하는 Super-Resolution(SR) 기법에 관심을 가지게 되었습니다.

그렇지만 몇 가지 제약이 있었습니다.
- 대규모 Paired Dataset 필요
- 새로운 환경(다른 현미경 조건)에서는 일반화 성능이 떨어짐
</p>

<blockquote><b>Introduction</b></blockquote>
<p>
이 논문은 Unpaired Dataset을 사용하여 Confocal Microscopy(공초점 현미경)의 Super-Resolution 문제를 해결하고자 합니다.  
핵심 아이디어는 아래와 같습니다.
- CycleGAN 프레임워크를 도입하여 Paired Dataset이 없는 상황에서도 학습 가능
- <b>Adversarial Loss, Cycle-Consistency Loss, Perceptual Loss</b>를 사용하여 현실성, 구조 보존, 안정적인 변환 달성
- VGG 19 Feature Map을 활용하여 구조적 세부 정보 유지
</p>

<blockquote><b>Method</b></blockquote>
<p>
모델 구조는 크게 세 부분으로 나눌 수 있습니다.
1. Generator : LR 영상을 HR 스타일로 변환
2. Discriminator : HR 이미지와 생성된 이미지 구분
3. Loss Functions : 
 - Adversarial Loss : 진짜/가짜 구분
 - Cycle-Consistency Loss : 변환 후 다시 원래 도메인으로 복원했을 때 차이를 최소화
 - Perceptual Loss : VGG 19 기반 구조적 특징 보존

 추가적으로, 이 모델의 성능을 평가하기 위해 Paired Dataset을 사용하여 PSNR, SSIM, MSE 외에도 <b>BGSTD(배경 노이즈 정량화), CUF(공간 차단 주파수 분석)</b> 같은 현미경 특화 지표를 도입했습니다.
</p>

<blockquote><b>Results</b></blockquote>
<figure>
  <img src="/assets/paper_img/0903/fig2.png" alt="Figure 2" style="max-width:100%; border-radius:12px;">
</figure>

<p>
- Unpaired Dataset 만으로도 Paired Supervised 방식에 근접하는 성능 달성
- CycleGAN 기반 Loss 조합을 통해 구조적 보존 + Artifact 억제 효과 확인
- BGSTD, CUF 등의 특수 메트릭에서도 우수한 결과 -> 단순 PSNR/SSIM 보다 더 실제 현미경 이미지 연구에 적합 
</p>
