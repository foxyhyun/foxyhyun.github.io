---
title: "Image Features"
course: computer-vision
course_title: "Computer Vision"
order: 4
description: "딥러닝 이전에 이미지를 어떻게 표현했는지, Feature Transform부터 HOG, Bag of Words까지 전통적인 이미지 특징 추출 방법을 살펴봅니다."
mathjax: true
---

## 왜 Image Features가 필요한가?

Linear Classifier는 원본 픽셀 값을 그대로 입력으로 받습니다. 하지만 픽셀 공간에서는 선형으로 분리하기 어려운 데이터가 많습니다. 예를 들어, 같은 물체라도 조명·각도·위치가 달라지면 픽셀 분포가 완전히 달라지기 때문입니다.

해결책은 **픽셀을 직접 쓰는 대신, 의미 있는 특징(feature)으로 변환**하는 것입니다. 좋은 feature representation은 모델이 풀어야 할 문제를 훨씬 쉽게 만들어줍니다.

---

## Feature Transform

### 아이디어: 공간을 바꾸면 선형 분리가 가능해진다

아래 예시를 보면 직관이 잡힙니다.

<figure>
  <img src="{{ '/assets/lecture_img/image_features/img1.png' | relative_url }}" alt="Feature Transform 시각화">
  <figcaption>원본 공간(왼쪽)에서는 비선형 경계가 필요하지만, 적절한 feature space(오른쪽)로 변환하면 선형 경계로 분리됩니다.</figcaption>
</figure>

왼쪽 원본 공간에서 파란 점들은 원형으로 분포하고 초록 점들이 가운데에 있습니다. 이 공간에서 두 그룹을 선형으로 나누는 것은 불가능합니다.

하지만 좌표를 다음과 같이 변환하면:

$$r = \sqrt{x^2 + y^2}, \quad \theta = \tan^{-1}(y/x)$$

각 점이 원점으로부터의 **거리(r)**와 **각도(θ)**로 표현됩니다. 이 feature space에서는 두 그룹이 $r$ 축을 기준으로 깔끔하게 선형 분리됩니다.

즉, **원본 공간에서 비선형 분류기**가 필요했던 문제가 **feature space에서는 선형 분류기**로 해결됩니다. 이것이 feature transform의 핵심 아이디어입니다.

<span style="color:#60a5fa">이 아이디어는 **kernel method**와도 연결됩니다. SVM의 kernel trick은 데이터를 명시적으로 고차원으로 변환하지 않고도 그 공간에서의 내적을 계산하는 방식으로, feature transform을 암묵적으로 수행합니다. 딥러닝에서도 각 레이어가 일종의 feature transform을 학습한다고 볼 수 있습니다. 앞단 레이어는 엣지나 텍스처 같은 저수준 특징을, 뒷단 레이어는 물체의 부분이나 전체 형태 같은 고수준 특징을 표현하도록 변환됩니다.</span>

---

## HOG (Histogram of Oriented Gradients)

### 아이디어: 엣지의 방향 분포로 물체를 표현한다

물체의 형태는 픽셀 값 자체보다 **엣지의 방향**으로 더 잘 표현됩니다. 같은 고양이 사진이라도 조명이 달라지면 픽셀 값은 크게 변하지만, 귀나 윤곽선의 엣지 방향은 크게 변하지 않습니다.

<figure>
  <img src="{{ '/assets/lecture_img/image_features/img2.png' | relative_url }}" alt="HOG 특징 추출 과정">
  <figcaption>이미지를 8×8 픽셀 셀로 나누고, 각 셀에서 gradient 방향의 히스토그램을 계산합니다. 320×240 이미지의 경우 40×30개의 셀, 셀당 9방향 → 10,800차원 feature vector.</figcaption>
</figure>

### 계산 과정

1. **각 픽셀에서 gradient 계산**: 픽셀 값의 변화량(방향과 크기)을 구합니다.
2. **8×8 셀로 분할**: 이미지를 작은 구역으로 나눕니다.
3. **각 셀에서 histogram 계산**: gradient 방향을 보통 9개의 bin(0°~180° 또는 0°~360°)으로 나누고, 각 방향의 gradient 크기를 누적합니다.
4. **블록 단위 정규화**: 인접한 셀들을 묶어 contrast 변화에 강인하게 정규화합니다.

예를 들어, 320×240 이미지를 8×8 셀로 나누면:
- 가로 방향 셀 수: 320 / 8 = **40개**
- 세로 방향 셀 수: 240 / 8 = **30개**
- 방향 bin 수: **9개**
- 최종 feature vector 크기: $40 \times 30 \times 9 = \mathbf{10,800}$

<span style="color:#60a5fa">HOG가 등장한 배경은 2005년 Dalal & Triggs의 보행자 탐지 논문입니다. 딥러닝 이전까지 보행자·사람 탐지 분야에서 오랫동안 표준으로 사용됐습니다. 방향 bin을 0°~180°(부호 없는 방향)로 할지 0°~360°(부호 있는 방향)로 할지는 문제에 따라 다릅니다. 엣지는 방향만 중요하고 어느 쪽에서 보느냐가 관계없을 때는 부호 없는 방향(9 bins)이 더 잘 동작합니다. 정규화 방식으로는 L2-norm이나 L2-Hys 등이 사용되며, 이웃한 셀들의 정보를 합쳐 블록을 구성하면 조명 변화에 더 강인해집니다.</span>

---

## Bag of Words (BoW)

### 아이디어: 이미지를 "시각 단어"의 빈도 분포로 표현한다

텍스트 분류에서 문서를 단어 빈도의 히스토그램으로 표현하는 "Bag of Words" 방법을 이미지에 적용한 것입니다. 이미지 안에 어떤 **시각 패턴(visual word)**이 얼마나 자주 등장하는지로 이미지를 표현합니다.

<figure>
  <img src="{{ '/assets/lecture_img/image_features/img3.png' | relative_url }}" alt="Bag of Words 파이프라인">
  <figcaption>Step 1에서 random patch를 추출해 클러스터링으로 codebook을 만들고, Step 2에서 각 이미지를 codebook 기준으로 인코딩합니다.</figcaption>
</figure>

### Step 1: Codebook 구축

1. 훈련 이미지에서 **random patch**를 대량으로 추출합니다.
2. 각 patch를 픽셀 값 또는 SIFT 등의 descriptor로 표현합니다.
3. **K-means 클러스터링**으로 유사한 patch들을 묶어 대표 패턴(cluster center)을 만듭니다.
4. 이 대표 패턴들이 "시각 단어(visual word)"로 이루어진 **codebook**이 됩니다.

### Step 2: 이미지 인코딩

1. 새 이미지에서 patch를 추출합니다.
2. 각 patch를 codebook의 어떤 visual word와 가장 가까운지 찾습니다(nearest neighbor).
3. **어떤 visual word가 몇 번 등장했는지** 히스토그램을 만듭니다.
4. 이 히스토그램이 해당 이미지의 feature vector입니다.

결과적으로 이미지의 크기나 물체의 위치와 무관하게, **어떤 패턴들이 얼마나 있는지**만으로 이미지를 표현하게 됩니다.

<span style="color:#60a5fa">Bag of Words는 **위치 정보를 무시**한다는 치명적인 단점이 있습니다. 하늘이 위에 있는 이미지와 아래에 있는 이미지가 같은 BoW 표현을 가질 수 있습니다. 이를 보완하기 위해 **Spatial Pyramid Matching(SPM)**이 제안됐습니다. 이미지를 1×1, 2×2, 4×4 등 여러 해상도의 격자로 나누고, 각 격자 셀마다 BoW 히스토그램을 계산한 뒤 이어 붙이는 방식입니다. 이렇게 하면 "어디에 무엇이 있는지"까지 일정 수준 표현할 수 있습니다. 또한 visual word의 수(K)는 보통 수백~수천 개를 사용하며, K가 너무 작으면 표현력이 부족하고 너무 크면 각 word가 너무 세분화되어 일반화가 어려워집니다.</span>

---

## Feature Engineering vs. Deep Learning

딥러닝 이전까지는 HOG, SIFT, BoW 같은 **hand-crafted feature**를 사람이 직접 설계하고, 그 위에 Linear SVM 같은 분류기를 학습시키는 것이 표준 파이프라인이었습니다.

```
이미지 → [Hand-crafted Feature 추출] → Feature Vector → [Linear Classifier] → 예측
          (사람이 설계)                                   (학습으로 결정)
```

딥러닝은 이 패러다임을 바꿨습니다.

```
이미지 → [CNN] → 예측
          (feature 추출 + 분류를 end-to-end로 함께 학습)
```

CNN은 feature를 사람이 설계하지 않고, **데이터로부터 자동으로 학습**합니다. 결과적으로 ImageNet 같은 대규모 데이터셋에서 hand-crafted feature를 압도하는 성능을 보였습니다.

<span style="color:#60a5fa">그렇다고 hand-crafted feature가 완전히 쓸모없어진 것은 아닙니다. 데이터가 극히 적거나, 해석 가능성(interpretability)이 중요한 의료·산업 분야에서는 여전히 HOG나 SIFT 기반 방법이 사용됩니다. 또한 딥러닝 모델이 내부적으로 학습하는 특징을 시각화해보면, 초기 레이어에서는 HOG와 유사한 엣지·방향 필터가 자연스럽게 학습된다는 것이 알려져 있습니다. 즉, hand-crafted feature는 딥러닝이 자동으로 발견하는 것을 사람이 먼저 직관적으로 찾아낸 것이라고도 볼 수 있습니다.</span>

---

## 핵심 질문

1. **Feature Transform의 핵심 아이디어는?**
원본 픽셀 공간에서는 비선형 경계가 필요한 문제도, 적절한 feature space로 변환하면 선형 분리가 가능해집니다. 문제를 더 쉽게 만드는 표현을 찾는 것이 핵심입니다.

2. **HOG의 계산 과정을 단계별로 설명하라.**
① 각 픽셀에서 gradient 방향과 크기 계산 → ② 8×8 셀로 분할 → ③ 각 셀에서 방향 histogram 계산(gradient 크기로 가중) → ④ 인접 셀들을 블록으로 묶어 정규화.

3. **320×240 이미지를 HOG로 표현하면 feature vector 크기는?**
$40 \times 30 \times 9 = 10,800$. (가로 40셀 × 세로 30셀 × 9방향 bin)

4. **Bag of Words의 두 단계는?**
① Codebook 구축: 훈련 이미지에서 patch를 추출해 K-means로 클러스터링 → visual word 사전 생성. ② 이미지 인코딩: 각 이미지의 patch를 nearest visual word에 할당하고 빈도 히스토그램 생성.

5. **Bag of Words의 가장 큰 단점과 이를 보완하는 방법은?**
위치 정보를 무시한다는 것이 단점입니다. Spatial Pyramid Matching(SPM)으로 이미지를 여러 해상도의 격자로 나눠 각 구역마다 BoW 히스토그램을 구하면 공간 정보를 부분적으로 반영할 수 있습니다.

6. **Hand-crafted feature와 딥러닝 CNN의 차이는?**
Hand-crafted feature는 사람이 직접 설계한 feature를 추출한 뒤 classifier를 별도로 학습합니다. CNN은 feature 추출과 분류를 end-to-end로 함께 학습하며, 데이터로부터 자동으로 최적의 feature를 찾아냅니다.

7. **HOG가 픽셀 값보다 조명 변화에 강인한 이유는?**
gradient 방향은 절대적인 밝기보다 픽셀 값의 **변화 방향**을 보기 때문입니다. 전체 조명이 밝아지거나 어두워지면 픽셀 값은 크게 달라지지만, 엣지의 방향과 상대적 강도 분포는 크게 변하지 않습니다.
