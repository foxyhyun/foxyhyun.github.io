---
title: "#2. Human vs Computer Vision System"
categories: [Tutorial]
tags: [ImageProcessing]
layout: post
mathjax: true
---

## Human vs Digital Image Processing

인간은 빛, 소리, 압력 같은 물리적 자극을 감각기관으로 받아들이고, 이를 뇌에서 해석해 시각, 청각, 촉각 같은 **지각(Perception)** 으로 변환합니다.  
즉, 단순 입력(Input)이 아니라 **Sensation(감각) → Process(해석) → Perception(지각)** 을 거쳐 인지가 이루어집니다.

![Perception]({{ '/assets/tutorial_img/20251015/fig1.png' | relative_url }})

인간의 눈은 카메라와 유사한 구조를 갖고 수용체를 통해 빛을 감지합니다.  
Digital Image Processing은 이 생물학적 시각 시스템을 모델 삼아 이미지 센서 데이터를 처리하고 인간의 인지 과정에 맞게 보정·향상하는 기술입니다.

---

인간의 눈은 단순히 빛의 세기를 물리적 수치로 측정하지 않고, **대비(Contrast)와 맥락(Context)** 에 따라 밝기를 다르게 지각하는 **비선형 시스템**입니다.  
우리가 보는 이미지는 단순한 픽셀 값이 아니라, **조명(illumination) × 반사(reflection)** 로 형성됩니다.

![Illumination vs Reflection]({{ '/assets/tutorial_img/20251015/fig2.png' | relative_url }})

또한 눈은 **HDR(High Dynamic Range)** 특성을 지녀, 어둠 속에서도 사물을 구분하고 강한 햇빛 아래에서도 적응할 수 있습니다.

---

## 영상 품질 요소

### Aperture
![Aperture]({{ '/assets/tutorial_img/20251015/fig3.png' | relative_url }})
- **Aperture가 크면**: 많은 빛이 들어오며 여러 방향의 빛이 평균화 → **Blurry**(DOF 얕아짐)
- **Aperture가 작으면**: 빛이 적게 들어와 어두워짐 + **회절(Diffraction)** 때문에 **Blurry**

### FOV(Field of View) vs DOF(Depth of Field)
![Aperture]({{ '/assets/tutorial_img/20251015/fig4.png' | relative_url }})
- **FOV**: 카메라가 얼마나 넓게 볼 수 있는가(렌즈 초점거리 + 센서 크기에 의해 결정)
- **DOF**: 카메라가 얼마나 깊이까지 선명하게 볼 수 있는가(조리개, 피사체와의 거리, 초점거리의 영향)

### Exposure를 결정하는 3 요소

1. **ISO**
![Aperture]({{ '/assets/tutorial_img/20251015/fig5.png' | relative_url }})
    - 낮을수록: 이미지가 깨끗함(노이즈 적음)
    - 높을수록: 어두운 환경에서도 촬영 가능하지만 노이즈 증가 및 다이내믹레인지 감소

2. **Shutter Speed**
![Aperture]({{ '/assets/tutorial_img/20251015/fig6.png' | relative_url }})
    - 빠를수록: 순간 포착 가능
    - 느릴수록: 노출 시간이 길어져 이미지가 밝아짐(모션 블러 증가)

3. **Aperture**
    - 조리개가 작을수록(숫자 \(N\) 큼): 빛이 적게 들어와 어두워짐, **DOF 깊어짐**
    - 조리개가 클수록(숫자 \(N\) 작음): 더 밝아지며 **DOF가 얕아짐**


