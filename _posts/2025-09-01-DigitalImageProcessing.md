---
title: "Digital Image Processing란?"
categories: [Tutorial]
tags: [ImageProcessing]
layout: post
---

**Digital Image Processing** 은 이미지를 더 잘 보이게 만드는 기술이다.

![Digital Image Processing Pipeline](assets/tutorial_img/20250901/fig1.png)

현실 **Scene -> Camera/Sensor -> Image -> Processing -> Processed Image** 로 이어지는 파이프라인에서, 센서는 **빛** 을 전기 신호로 바꾸고, 처리 단계에서는 **필터링·향상·복원·변환**이 이루어진다. 이때 핵심 배경은 **전자기 스펙트럼**이다. **가시광(약 400–700 nm)** 을 포함해 파장대가 달라지면 **센서·조명·얻는 정보의 성격**이 달라지므로, **목적과 파장 특성에 맞춘 처리**가 필요하다.

스펙트럼 관점에서 영상 응용을 보면 직관이 생긴다. **감마선/X-ray** 는 **의료·보안**, **적외선(IR)** 은 **열 정보·야간 감시**, **멀티/하이퍼스펙트럴** 은 **농업·원격탐사**, **현미경 영상**은 **반도체/세포** 같은 미세 구조 관찰, **MRI** 는 **연부조직**에 강하다.  
→ **즉, “어떤 파장대를 쓰느냐 = 어떤 물리 정보를 볼 것이냐”** 에 가깝다.

![rays-to-color](assets/tutorial_img/20250901/fig2.png)

이미지는 **광선(ray)** 을 **색/밝기**로 매핑하는 함수로 볼 수 있다.  
$$f(\mathbf{r}) = \mathbf{c}$$  
센서 평면 위에서는 **좌표 \((x, y)\)** 에 대한 **2차원 함수**로 표현한다.  
$$f(x,y) \rightarrow \text{intensity / color}$$

![image-to-pixel](assets/tutorial_img/20250901/fig3.png)

연속적인 장면을 **이산 샘플링(discrete sampling)** 하면 **픽셀(pixel)**이 된다.  
픽셀은 **picture element**의 줄임말로, **한 좌표의 샘플 값**을 뜻한다.


![pixel representation](assets/tutorial_img/20250901/fig4.png)
실제 메모리에서는 보통 **행(row)–열(column)** 순서의 2D 배열(혹은 3D 텐서)로 저장된다.  
- **그레이스케일**: 한 픽셀 = **단일 강도값**  
- **컬러(RGB)**: 한 픽셀 = **3채널 값**  
- **표현 범위**는 크게 두 가지가 흔하다  
  - **실수형 강도** \(I \in \mathbb{R}\): 보통 **정규화 \([0,1]\)** 또는 **영점 중심 \([-1,1]\)**  
  - **정수형(uint8)**: **\[0, 255\]** (1 byte)