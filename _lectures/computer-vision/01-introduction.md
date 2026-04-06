---
title: "Introduction to Computer Vision"
course: computer-vision
course_title: "Computer Vision"
order: 1
description: "Computer Vision이란 무엇인지, 어떤 문제를 풀고 어떻게 활용되는지 개요를 살펴봅니다."
mathjax: false
---

## Computer Vision이란?

**Computer Vision(컴퓨터 비전)**은 컴퓨터가 디지털 이미지나 영상으로부터 의미 있는 정보를 추출하고 이해하도록 만드는 AI 분야입니다.

인간은 눈을 통해 세상을 보고 뇌가 즉시 물체를 인식하지만, 컴퓨터에게 이미지는 그저 숫자의 배열(픽셀 값)에 불과합니다. Computer Vision은 이 숫자 배열에서 "고양이가 있다", "사람이 걸어간다", "신호등이 빨간색이다" 같은 정보를 자동으로 뽑아내는 기술입니다.

---

## 주요 태스크

| 태스크 | 설명 | 예시 |
|---|---|---|
| **Image Classification** | 이미지를 하나의 클래스로 분류 | 개 vs 고양이 |
| **Object Detection** | 이미지 내 여러 객체의 위치와 클래스를 동시에 | YOLO, Faster R-CNN |
| **Semantic Segmentation** | 픽셀 단위로 클래스 분류 | 도로 / 건물 / 하늘 |
| **Instance Segmentation** | 동일 클래스도 개체별로 구분 | Mask R-CNN |
| **Image Generation** | 새로운 이미지를 합성 | GAN, Diffusion |
| **Optical Flow** | 프레임 간 물체의 움직임 추정 | 자율주행 |

---

## 왜 어려운가?

컴퓨터 비전이 어려운 이유는 **같은 물체도 보는 조건에 따라 완전히 다르게 보이기** 때문입니다.

- **조명 변화**: 밝은 낮 vs 어두운 밤의 같은 장면
- **시점 변화**: 정면 / 측면 / 위에서 본 같은 물체
- **변형(Deformation)**: 구부러진 고양이, 뛰는 사람
- **폐색(Occlusion)**: 다른 물체에 가려진 경우
- **배경 혼잡(Clutter)**: 복잡한 배경 속 물체

---

## 역사 흐름

```
1960s  → 초기 edge detection, 블록 세계 인식
1980s  → 수작업 feature 기반 방법 (SIFT, HOG 등)
2012   → AlexNet — Deep Learning 시대 개막
2015~  → ResNet, VGG 등 심층 CNN 발전
2017~  → Attention, Transformer 도입
2020~  → ViT (Vision Transformer), CLIP, Diffusion Model
```

> AlexNet이 ImageNet 대회에서 압도적 성능으로 우승한 2012년을 흔히 **딥러닝 CV의 원년**으로 부릅니다.

---

## 다음 강의 예고

다음 시간에는 이미지가 컴퓨터 내부에서 어떻게 표현되는지, **픽셀, 채널, 해상도** 등 기초 개념을 다룹니다.
