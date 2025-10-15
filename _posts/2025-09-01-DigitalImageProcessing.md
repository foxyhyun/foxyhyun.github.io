---
title: "#1. Image Processing이란?"
categories: [Tutorial]
tags: [ImageProcessing]
layout: post
mathjax: true
---

**Digital Image Processing** 은 이미지를 <mark>더 잘 보이게</mark> 만드는 기술이다.

![DIP pipeline]({{ '/assets/tutorial_img/20250901/fig1.png' | relative_url }})

현실 **Scene → Camera/Sensor → Image → Processing → Processed Image** 로 이어지는 파이프라인에서, 센서는 **빛**을 전기 신호로 바꾸고, 처리 단계에서는 **필터링 · 향상 · 복원 · 변환**이 이루어진다. 이때 핵심 배경은 **전자기 스펙트럼**이다. **가시광(약 400–700 nm)** 을 포함해 파장대가 달라지면 **센서 · 조명 · 얻는 정보의 성격**이 달라지므로, **목적과 파장 특성에 맞춘 처리**가 필요하다.

스펙트럼 관점에서 영상 응용을 보면 직관이 생긴다. **감마선/X-ray** 는 **의료·보안**, **적외선(IR)** 은 **열 정보·야간 감시**, **멀티/하이퍼스펙트럴** 은 **농업·원격탐사**, **현미경 영상**은 **반도체/세포** 같은 미세 구조 관찰, **MRI** 는 **연부조직**에 강하다.  
→ 즉, “<mark>어떤 파장대를 쓰느냐 = 어떤 물리 정보를 볼 것이냐</mark>” 에 가깝다.

![rays to color]({{ '/assets/tutorial_img/20250901/fig2.png' | relative_url }})

이미지는 **광선(ray)** 을 **색/밝기**로 매핑하는 함수로 볼 수 있다.  
$$f(\mathbf{r})=\mathbf{c}$$  
센서 평면 위에서는 **좌표 \((x,y)\)** 에 대한 **2차원 함수**로 표현한다.  
$$f(x,y)\rightarrow \text{intensity / color}$$

![image to pixel]({{ '/assets/tutorial_img/20250901/fig3.png' | relative_url }})

연속적인 장면을 **이산 샘플링(discrete sampling)** 하면 **픽셀(pixel)** 이 된다.  
픽셀은 **picture element**의 줄임말로, **한 좌표의 샘플 값**을 뜻한다.

![pixel representation]({{ '/assets/tutorial_img/20250901/fig4.png' | relative_url }})

실제 메모리에서는 보통 **행(row)–열(column)** 순서의 2D 배열(혹은 3D 텐서)로 저장된다.  
- **그레이스케일**: 한 픽셀 = **단일 강도값**  
- **컬러(RGB)**: 한 픽셀 = **3채널 값**  
- **표현 범위**(예시)  
  - **실수형 강도** $I \in \mathbb{R}$ : **정규화 \([0,1]\)** 또는 **영점 중심 \([-1,1]\)**  
  - **정수형(uint8)**: **\[0,255\]** (1 byte)

---

## Image Processing 맛보기

1) Image Restoration

![Denoising before/after]({{ '/assets/tutorial_img/20250901/fig5.png' | relative_url }})


![Deblurring before/after]({{ '/assets/tutorial_img/20250901/fig6.png' | relative_url }})


---

2) Image Enhancement

![Exposure & contrast]({{ '/assets/tutorial_img/20250901/fig7.png' | relative_url }})


![Sharpening]({{ '/assets/tutorial_img/20250901/fig8.png' | relative_url }})


---

3) Image Compression

![Compression rate-PSNR trade-off]({{ '/assets/tutorial_img/20250901/fig9.png' | relative_url }})

---

4) Edges, Corners, Features

![Edge detection]({{ '/assets/tutorial_img/20250901/fig10.png' | relative_url }})

- **Edges:** 강한 기울기(경계) 추출 → **윤곽/분할/특징**의 기초

![Corner detection]({{ '/assets/tutorial_img/20250901/fig11.png' | relative_url }})

- **Corners:** 회전/조명 변화에 비교적 강인 → **정합/추적/SLAM** 핵심 키포인트

![Local features & descriptors]({{ '/assets/tutorial_img/20250901/fig12.png' | relative_url }})

- 로컬 패치 → 벡터화 → **매칭/정합**. 전통 특징 ↔ 학습기반(SuperPoint, R2D2 등)

---

5) Image Representation

![Multi-scale / orientation responses]({{ '/assets/tutorial_img/20250901/fig13.png' | relative_url }})

---

## Human vs Digital Image Processing

인간은 빛, 소리, 압력 같은 물리적 자극을 감각기관으로 받아들이고, 이를 뇌에서 해석해 시각, 청각, 촉각 같은 **지각(Perception)** 으로 변환합니다.  
즉, 단순 입력(Input)이 아니라 **Sensation(감각) → Process(해석) → Perception(지각)** 이라는 과정을 거쳐 인지가 이루어집니다.   
  
인간의 눈은 카메라와 유사한 구조를 갖고 수용체를 통해 빛을 감지합니다.  
Digital Image Processing은 이 생물학적 시각 시스템을 모델 삼아 이미지 센서 데이터를 처리하고 인간의 인지 과정에 맞게 보정, 향상하는 기술입니다. 

---

인간의 눈은 단순히 빛의 세기를 물리적 수치로 측정하지 않고, **대비(Contrast)와 맥락(Context)**에 따라 밝기를 다르게 지각하는 **비선형 시스템**입니다.  
우리가 보는 이미지는 단순한 픽셀 값이 아니라, **조명(illumination) X 반사(reflection)** 으로 형성됩니다.  
또한 눈은 **HDR(High Dynamic Range)** 특성을 지녀, 어둠 속에서도 사물을 구분하고 강한 햇빛 아래에서도 적응할 수 있습니다.  

---

디지털 이미지는 실제 세상에서 오는 빛을 **Sampling**과 **Quantization** 과정을 거쳐 얻은 픽셀 값의 집합입니다. 
- **Sampling** : 
- **Quantization** : 
이는 수학적으로 2D 함수 f(x,y)로 표현되며, Digital Image Processing은 이 픽셀 함수에 다양한 수학적 연산을 적용해 더 나은 시각적 결과를 만들어내는 과정입니다.  

---
## 영상 품질 요소

### Aperture  
- Aperture가 크면 : 많은 빛이 들어오면서 여러 방향의 빛이 평균화 → Blurry
- Aperture가 작으면 : 빛이 적게 들어와 어두워짐 + Diffraction(빛이 좁은 틈을 통과하며 퍼지는 현상) 때문에 Blurry  

### FOV(Field of View) vs DOF(Depth of Field)  
- **FOV** : 카메라가 얼마나 넓게 볼 수 있는가(렌즈 초점거리 + 센서 크기에 의해 결정)  
- **DOF** : 카메라가 얼마나 깊이까지 선명하게 볼 수 있는가(조리개, 피사체와의 거리, 초점거리의 영향)  

### Exposure을 결정하는 3 요소  
1. ISO  
  - 낮을수록 : 이미지가 깨끗함(노이즈 적음)  
  - 높을수록 : 어두운 환경에서도 촬영 가능하지만 노이즈 증가  

2. Shutter Speed  
  - 빠를수록 : 순간 포착 가능  
  - 느릴수록 : 노출 시간이 길어져 이미지가 밝아짐  

3. Aperture  
  - 조리개가 작을수록 : 빛이 적게 들어와 어두워짐
  - 조리개가 클수록 : 더 밝아지면 DOF가 얉아짐  

