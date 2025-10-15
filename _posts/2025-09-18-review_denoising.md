---
title: "[Bioimage Denoising] Imaging in focus: An introduction to denoising bioimages in the era of deep learning"
categories: [PaperReview]
tags: [Denoising, Bioimaging, DeepLearning, Review]
year: "2021"
venue: "International Journal of Biochemistry and Cell Biology"
layout: post
paper: "https://www.sciencedirect.com/science/article/pii/S1357272521001588"
---

## Abstract  
**형광 현미경 이미지**는 조명 독성과 저광량 촬영으로 인해 심한 노이즈가 발생하기 쉽다. 이에 따라 최근에는 명시적 수학 모델 대신 데이터로부터 구조와 내용 맥락을 학습하는(**content-aware**) **딥러닝 기반 디노이징**이 활발히 사용되고 있다. 본 리뷰는 형광 영상에 흔한 노이즈 유형을 체계적으로 정리하고, 그에 대응하는 디노이징 과제 정의를 제시한 뒤, 딥러닝 기반 방법들의 원리와 장단점을 비교해 사용자 응용별 선택 가이드를 제공한다.  
  
**Fluorescence microscopy images** often suffer from severe noise due to illumination toxicity and low-light acquisition. Consequently, content-aware deep learning approaches that learn structure and context from data—rather than relying on explicit mathematical models—have gained traction. This review surveys common noise types in fluorescence imaging, formalizes the denoising task, and compares DL-based methods with their advantages and limitations, providing a practical guide for choosing methods by application.  

## Introduction  
형광 현미경 이미지는 노이즈를 피하기 어렵다. 특히 **라이브 셀 이미징**에서는 **광독성(phototoxicity)** 과 **표백(photobleaching)** 을 줄이기 위해 저조도·고속 촬영을 사용하므로, **포톤 샷 노이즈(대개 포아송)**와 **카메라 읽기 노이즈(가우시안)** 가 크게 나타난다.
획득 이후(post-acquisition)에는 노이즈를 줄여 세그멘테이션, 정량 분석 같은 다운스트림 작업의 신뢰도를 높이는 것이 핵심 과제다.  

노이즈 제거의 가장 단순한 접근은 **스무딩 필터**(예: 가우시안)지만, 구조를 흐릴 수 있다는 한계가 있다. 이 문제를 완화하기 위해 고전적 비학습 방법들이 제안되었는데, 대표적으로 **Non-Local Means (NLM), BM3D (Block-Matching 3D), 웨이블릿 변환 기반 감쇠(shrinkage)** 등이 있다.  

최근에는 **머신러닝**, 특히 딥러닝이 현미경 디노이징의 주류로 자리잡았다. **CARE(Content-Aware Restoration), (3D)RCAN 계열, Noise2Void** 와 같은 방법들은 데이터로부터 내용 인지(**content-aware**)적으로 구조를 학습하여 노이즈를 제거하며, 많은 경우 **SSIM/PSNR** 등 정량 지표와 세포 경계·소기관 보존에서 고전적 방법보다 우수한 성능을 보인다.

Fluorescence microscopy images are inherently noisy. In **live-cell imaging**, low illumination and high frame rates are used to mitigate **phototoxicity** and **photobleaching**, which in turn amplify **photon shot noise** (often Poisson) and **camera read noise** (Gaussian). After acquisition, denoising is crucial to enable reliable downstream analyses such as segmentation and quantitative measurements.

The simplest approach is **smoothing** (e.g., Gaussian filtering), but it tends to blur structures. Classical is the non-learning methods including **Non-Local Means (NLM), BM3D, and wavelet transform**.

More recently, machine learning, especially deep learning, has become the dominant paradigm. Methods such as **CARE (Content-Aware Restoration), (3D)RCAN-based models, and self-supeㅠrvised approaches like Noise2Void** learn **content-aware** priors from data, typically achieving higher **SSIM/PSNR** and better preservation of cellular structures than classical techniques.

## Noise and the task of denoising  

So what is Noise?  
모든 형광 이미지는 다양한 요소에 의해 noise, out-of-focus, artefact등이 생길 수 있다. 
대부분의 노이즈는 shot noise와 detector noise이다.  
Shot noise는 자연광에 의한 노이즈이며 수학적으로 보면 포아송 분포를 따른다. which scales with intensity of the signal. this means bright pixels exhibit a larger noise level compared to darker ones. (이 부분 정리)
Detector noise는 보통 각각 픽셀에 독립적이고 규칙적으로 나타난다. 종종 간단한 가우시안 노이즈를 더하는 과정이다.  
