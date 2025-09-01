---
title: "Digital Image Processing란?"
categories: [Tutorial]
tags: [ImageProcessing]
layout: post
---

**Digital Image Processing** 은 이미지를 더 잘 보이게 만드는 기술이다.

<div align="center">
  <img src="/assets/tutorial_img/20250901/fig1.png" alt="Multi-scale/Orientation filter responses" width="72%">
  <br><sub><b>DPI pipeline</sub>
</div>

현실 **Scene -> Camera/Sensor -> Image -> Processing -> Processed Image** 로 이어지는 파이프라인에서, 센서는 **빛** 을 전기 신호로 바꾸고, 처리 단계에서는 **필터링·향상·복원·변환**이 이루어진다. 이때 핵심 배경은 **전자기 스펙트럼**이다. **가시광(약 400–700 nm)** 을 포함해 파장대가 달라지면 **센서·조명·얻는 정보의 성격**이 달라지므로, **목적과 파장 특성에 맞춘 처리**가 필요하다.

스펙트럼 관점에서 영상 응용을 보면 직관이 생긴다. **감마선/X-ray** 는 **의료·보안**, **적외선(IR)** 은 **열 정보·야간 감시**, **멀티/하이퍼스펙트럴** 은 **농업·원격탐사**, **현미경 영상**은 **반도체/세포** 같은 미세 구조 관찰, **MRI** 는 **연부조직**에 강하다.  
→ **즉, “어떤 파장대를 쓰느냐 = 어떤 물리 정보를 볼 것이냐”** 에 가깝다.

<div align="center">
  <img src="/assets/tutorial_img/20250901/fig2.png" alt="Multi-scale/Orientation filter responses" width="72%">
  <br><sub><b>rays to color</sub>
</div>


이미지는 **광선(ray)** 을 **색/밝기**로 매핑하는 함수로 볼 수 있다.  
$$f(\mathbf{r}) = \mathbf{c}$$  
센서 평면 위에서는 **좌표 \((x, y)\)** 에 대한 **2차원 함수**로 표현한다.  
$$f(x,y) \rightarrow \text{intensity / color}$$

<div align="center">
  <img src="/assets/tutorial_img/20250901/fig3.png" alt="Multi-scale/Orientation filter responses" width="72%">
  <br><sub><b>image to pixel</sub>
</div>


연속적인 장면을 **이산 샘플링(discrete sampling)** 하면 **픽셀(pixel)**이 된다.  
픽셀은 **picture element**의 줄임말로, **한 좌표의 샘플 값**을 뜻한다.


<div align="center">
  <img src="/assets/tutorial_img/20250901/fig4.png" alt="Multi-scale/Orientation filter responses" width="72%">
  <br><sub><b>pixel representation</sub>
</div>

실제 메모리에서는 보통 **행(row)–열(column)** 순서의 2D 배열(혹은 3D 텐서)로 저장된다.  
- **그레이스케일**: 한 픽셀 = **단일 강도값**  
- **컬러(RGB)**: 한 픽셀 = **3채널 값**  
- **표현 범위**는 크게 두 가지가 흔하다  
  - **실수형 강도** \(I \in \mathbb{R}\): 보통 **정규화 \([0,1]\)** 또는 **영점 중심 \([-1,1]\)**  
  - **정수형(uint8)**: **\[0, 255\]** (1 byte)

## Image Processing 맛보기

### 1) Image Restoration

<div align="center">
  <img src="/assets/tutorial_img/20250901/fig5.png" alt="Denoising before/after" width="88%">
  <br><sub><b>Denoising</b> — 노이즈 억제(감도↑, 디테일 손실 최소화가 관건)</sub>
</div>

- **Denoising:** 저조도/고ISO에서 필수. 전통(NL-Means/BM3D) ↔ 학습기반(DnCNN/Restormer 등)

<div align="center" style="margin-top:10px;">
  <img src="/assets/tutorial_img/20250901/fig6.png" alt="Deblurring before/after" width="88%">
  <br><sub><b>Deblurring</b> — 모션/포커스 블러 복원(커널 추정·비선형 최적화/딥러닝)</sub>
</div>

- **Deblurring:** **팬닝/손떨림/초점이탈** 복원. 커널 추정, 비선형 역문제, 최근엔 **blind deblurring** 딥러닝

---

### 2) Image Enhancement

<div align="center">
  <img src="/assets/tutorial_img/20250901/fig7.png" alt="Exposure/contrast enhancement examples" width="88%">
  <br><sub><b>노출·대비 향상</b> — 감마/히스토그램/Retinex/LLIE 계열</sub>
</div>

- **노출/대비 강화:** 감마 보정, HE/CLAHE, Retinex, 저조도 향상(LLIE) 등

<div align="center" style="margin-top:10px;">
  <img src="/assets/tutorial_img/20250901/fig8.png" alt="Sharpening before/after" width="88%">
  <br><sub><b>Sharpening</b> — 언샤프 마스크/고주파 강화(노이즈 증폭 주의)</sub>
</div>

- **Sharpening:** 고주파 성분 강조(엣지 또렷). 노이즈와 링잉(halo) 제어가 핵심

---

### 3) Image Compression

<div align="center">
  <img src="/assets/tutorial_img/20250901/fig9.png" alt="Compression rate-PSNR trade-off" width="88%">
  <br><sub><b>용량 ↔ 품질</b> 트레이드오프: 3.21 kB / 29.34 dB → 10.46 kB / 38.70 dB</sub>
</div>

- **JPEG/HEIF/WebP/AVIF** 등. 전형적으로 **변환(예: DCT/DWT)** → **양자화** → **엔트로피 부호화**
- **PSNR/SSIM/MS-SSIM** 등으로 품질 평가(시각적 품질과 1:1 대응은 아님)

---

### 4) Edges, Corners, Features

<div align="center">
  <img src="/assets/tutorial_img/20250901/fig10.png" alt="Edge detection" width="88%">
  <br><sub><b>Edge Detection</b> — Sobel/LoG/Canny(비최대 억제+히스테리시스)</sub>
</div>

- **Edges:** 강한 기울기(경계) 추출 → **윤곽, 분할, 특징**의 기초

<div align="center" style="margin-top:10px;">
  <img src="/assets/tutorial_img/20250901/fig11.png" alt="Corner detection" width="88%">
  <br><sub><b>Corner Detection</b> — Harris, FAST 등(방향 변화가 큰 지점)</sub>
</div>

- **Corners:** 회전/조명 변화에 비교적 강인 → **정합, 추적, SLAM** 등 핵심 키포인트

<div align="center" style="margin-top:10px;">
  <img src="/assets/tutorial_img/20250901/fig12.png" alt="Local features & descriptors" width="88%">
  <br><sub><b>Features & Descriptors</b> — SIFT/SURF/ORB…(스케일·회전 불변 로컬 기술자)</sub>
</div>

- **Descriptors:** 로컬 패치 → 벡터화 → **매칭/정합**. 전통 특징 ↔ 학습기반(SuperPoint, R2D2 등)

---

### 5) Image Representation

<div align="center">
  <img src="/assets/tutorial_img/20250901/fig13.png" alt="Multi-scale/Orientation filter responses" width="72%">
  <br><sub><b>멀티스케일·방향 응답</b> 예 — 라플라시안/도그(DoG)/가보 필터 등</sub>
</div>

- 이미지는 **멀티스케일 필터 뱅크**나 **주파수/라플라시안 피라미드**로 표현 가능  
- 분해 표현은 **복원/합성/압축/특징 추출**의 공통 토대

---

