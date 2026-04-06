---
title: "Linear Classification"
course: basic-ai
course_title: "Basic of AI"
order: 1
description: "Image Classification의 한계를 넘어 Parametric Approach를 도입하고, SVM Loss와 Softmax로 학습 가능한 분류기를 만드는 방법을 다룹니다."
mathjax: true
---

## Image Classification의 어려움

딥러닝 이전에도 이미지 분류는 어려운 문제였습니다. 주요 도전 과제는 다음과 같습니다.

- **Viewpoint variation** — 같은 물체라도 보는 각도에 따라 픽셀이 완전히 달라짐
- **Illumination** — 조명 변화에 민감
- **Occlusion** — 물체 일부가 가려짐
- **Deformation** — 물체 자체가 형태를 바꿈
- **Intraclass variation** — 같은 클래스 안에서도 모양이 다양함
- **Background clutter** — 배경과 구분이 어려움

---

## 전통적 방법 vs. 머신러닝

| 접근 방식 | 설명 |
|---|---|
| **Traditional** | Hand-crafted feature + 복잡한 알고리즘 |
| **ML** | 데이터 수집 → 분류기 학습 → 새 이미지에 평가 |

---

## K-Nearest Neighbor (KNN)

가장 단순한 분류기. 훈련 이미지 전체를 메모리에 저장해두고, 새 이미지가 들어오면 거리를 계산해 가장 가까운 K개의 레이블로 결정합니다.

$$d_1(I_1, I_2) = \sum_p |I_1^p - I_2^p| \quad \text{(L1 distance)}$$

$$d_2(I_1, I_2) = \sqrt{\sum_p (I_1^p - I_2^p)^2} \quad \text{(L2 distance)}$$

- L1은 축 방향에 sharp한 경계, L2는 circular한 경계를 만들어냄
- **KNN의 핵심**: Training은 느려도 되지만 Inference는 빨라야 한다 → KNN은 정반대라서 실제론 안 씀

### Hyperparameter 설정

- K, 거리 함수 등 직접 시도해봐야 함
- **Train / Val / Test split** 을 유지하고 val로 선택, test는 최종 1회만 사용
- **Cross-validation**: 데이터를 K-fold로 나눠 돌아가며 val로 사용 → 소규모 데이터셋에 유용

### Curse of Dimensionality

차원이 증가하면 공간을 채우는 데 필요한 데이터 수가 지수적으로 폭발합니다. 64×64 이미지 = 4096차원 → KNN은 고차원 이미지에 사실상 사용 불가.

---

## Linear Classification

### Parametric Approach

KNN과 달리 학습된 파라미터(W, b)를 이용해 추론합니다.

$$f(x, W) = Wx + b$$

- **W** (weight matrix): 학습으로 결정
- **b** (bias): 입력 X와 독립적인 classifier 고유 성질. b가 없으면 decision boundary가 원점을 반드시 지나야 하지만, b가 있으면 자유롭게 평행이동 가능

---

## Loss Function

### Multiclass SVM Loss (Hinge Loss)

현재 classifier가 얼마나 좋은지 수치화합니다.

$$L_i = \sum_{j \neq y_i} \max(0,\ s_j - s_{y_i} + 1)$$

$$L = \frac{1}{N} \sum_i L_i$$

자주 나오는 질문들:

| 질문 | 답 |
|---|---|
| 정답 클래스 score가 0.5 감소하면? | Loss 동일 (margin 내에 있으면 변화 없음) |
| Loss의 최솟값/최댓값? | 0 ~ ∞ |
| W가 작아 모든 s ≈ 0이면? | C − 1 (클래스 수 − 1) |
| j = y_i도 포함하면? | Loss + 1 증가 |

Squared Hinge Loss를 쓰면 조금 틀린 것보다 많이 틀린 것에 훨씬 큰 벌점을 줍니다.

---

## Softmax Classifier

SVM이 score의 상대적 크기만 보는 반면, Softmax는 확률로 해석합니다.

$$p_j = \frac{e^{s_j}}{\sum_k e^{s_k}}$$

$$L_i = -\log\left(\frac{e^{s_{y_i}}}{\sum_k e^{s_k}}\right)$$

왜 negative log를 쓰냐? 정답 클래스의 확률이 1에 가까울수록 loss가 0이 되길 원하기 때문입니다.

| 질문 | 답 |
|---|---|
| Loss 최솟값/최댓값? | 0 ~ ∞ |
| 초기화 시 모든 s ≈ 0이면? | log(C) |

---

## Regularization

Loss만 최소화하면 overfitting이 생길 수 있습니다. Regularization term을 추가해 W가 너무 커지는 걸 방지합니다.

$$L = \underbrace{\frac{1}{N} \sum_i L_i}_{\text{data loss}} + \underbrace{\lambda R(W)}_{\text{regularization}}$$

- **목적**: Overfitting 방지 + 최적화 안정성 향상
- 대표적인 방법: L2 regularization (weight decay), L1, Dropout 등