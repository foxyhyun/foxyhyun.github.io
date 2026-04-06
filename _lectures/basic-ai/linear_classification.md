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

<span style="color:#60a5fa">이 도전 과제들이 어려운 근본 이유는 컴퓨터가 이미지를 "픽셀 값의 숫자 배열"로만 보기 때문입니다. 같은 고양이 사진이라도 조명이 달라지면 픽셀 값이 전혀 다른 배열이 됩니다. 이 간극을 **Semantic Gap**이라고 부릅니다.</span>

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

<span style="color:#60a5fa">더 정확히는, KNN의 시간 복잡도는 Training O(1), Inference O(N)입니다. 반면 딥러닝 모델은 Training O(N)이지만 Inference는 파라미터만 저장하면 O(1)에 가깝습니다. L1 vs L2 선택 기준은 좌표계의 의미에 달려 있습니다. 특징 벡터의 각 차원이 독립적인 의미를 가질 때는 L1, 차원 간 관계가 중요할 때는 L2가 유리합니다.</span>

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

<figure>
  <img src="{{ '/assets/lecture_img/linear_classification/img1.png' | relative_url }}" alt="Parametric Approach 다이어그램">
  <figcaption>32×32×3 이미지(3072차원)를 W(10×3072)와 곱하고 b(10×1)를 더해 10개의 클래스 점수를 출력</figcaption>
</figure>

<span style="color:#60a5fa">W의 각 행(row)은 해당 클래스에 대한 **템플릿(template)**으로 해석할 수 있습니다. 예를 들어 "자동차" 행은 자동차처럼 생긴 픽셀 패턴을 표현하고, 입력 이미지와 내적(dot product)이 클수록 해당 클래스와 유사하다고 판단합니다. 이것이 Linear Classifier가 "템플릿 매칭"이라고 불리는 이유입니다.</span>

---

## Loss Function

### Multiclass SVM Loss (Hinge Loss)

현재 classifier가 얼마나 좋은지 수치화합니다.

$$L_i = \sum_{j \neq y_i} \max(0,\ s_j - s_{y_i} + 1)$$

$$L = \frac{1}{N} \sum_i L_i$$

아래 예시에서 cat(정답), car, frog 3개의 클래스에 대해 SVM Loss를 계산합니다.

<figure>
  <img src="{{ '/assets/lecture_img/linear_classification/img2.png' | relative_url }}" alt="SVM Loss cat 예시">
  <figcaption>Cat 클래스가 정답일 때: max(0, 5.1−3.2+1) + max(0, −1.7−3.2+1) = 2.9</figcaption>
</figure>

<figure>
  <img src="{{ '/assets/lecture_img/linear_classification/img3.png' | relative_url }}" alt="SVM Loss frog 예시">
  <figcaption>Frog 클래스가 정답일 때: 정답 score가 이미 충분히 높아 Loss = 0</figcaption>
</figure>

<figure>
  <img src="{{ '/assets/lecture_img/linear_classification/img4.png' | relative_url }}" alt="SVM Loss car 예시">
  <figcaption>Car 클래스가 정답일 때 Loss 계산</figcaption>
</figure>

<figure>
  <img src="{{ '/assets/lecture_img/linear_classification/img5.png' | relative_url }}" alt="SVM Loss 평균">
  <figcaption>전체 평균 Loss: L = (2.9 + 0 + 12.9) / 3 = 5.27</figcaption>
</figure>

자주 나오는 질문들:

| 질문 | 답 |
|---|---|
| 정답 클래스 score가 0.5 감소하면? | Loss 동일 (margin 내에 있으면 변화 없음) |
| Loss의 최솟값/최댓값? | 0 ~ ∞ |
| W가 작아 모든 s ≈ 0이면? | C − 1 (클래스 수 − 1) |
| j = y_i도 포함하면? | Loss + 1 증가 |

<span style="color:#60a5fa">margin을 1로 설정하는 것은 사실 임의적입니다. W의 스케일을 조정하면 margin도 달라지기 때문에, 중요한 것은 margin의 절대적 크기가 아니라 **정답 클래스가 오답보다 얼마나 더 높은 score를 가져야 하는가**라는 상대적 여유입니다. Squared Hinge Loss $\max(0, \cdot)^2$를 사용하면 조금 틀린 것보다 많이 틀린 예시에 훨씬 강한 패널티를 부여해 더 가파른 최적화가 가능합니다.</span>

---

## Softmax Classifier

SVM이 score의 상대적 크기만 보는 반면, Softmax는 확률로 해석합니다.

$$p_j = \frac{e^{s_j}}{\sum_k e^{s_k}}$$

$$L_i = -\log\left(\frac{e^{s_{y_i}}}{\sum_k e^{s_k}}\right)$$

왜 negative log를 쓰냐? 정답 클래스의 확률이 1에 가까울수록 loss가 0이 되길 원하기 때문입니다.

<figure>
  <img src="{{ '/assets/lecture_img/linear_classification/img6.png' | relative_url }}" alt="Softmax 확률 변환 과정">
  <figcaption>raw score → exp → normalize → 확률. 정답(cat)의 확률이 1.00에 가까울수록 Loss → 0</figcaption>
</figure>

| 질문 | 답 |
|---|---|
| Loss 최솟값/최댓값? | 0 ~ ∞ |
| 초기화 시 모든 s ≈ 0이면? | log(C) |

<span style="color:#60a5fa">초기화 시 loss가 log(C)인 이유: s ≈ 0이면 모든 클래스의 확률이 1/C로 균등해지고, 정답 확률도 1/C이므로 $-\log(1/C) = \log(C)$가 됩니다. 학습 초기에 loss가 log(C)근처인지 확인하는 것은 구현이 올바른지 검증하는 유용한 sanity check입니다. 또한 수치 안정성을 위해 실제 구현에서는 exp를 계산하기 전에 모든 score에서 최댓값을 빼줍니다: $e^{s_j - \max s}$. 이렇게 해도 확률값은 동일하지만 overflow를 방지할 수 있습니다.</span>

### SVM vs. Softmax 비교

| | SVM | Softmax |
|---|---|---|
| 출력 해석 | 상대적 score | 확률 |
| Loss | Hinge Loss | Cross-Entropy |
| 정답 score가 충분히 높으면 | Loss = 0, 무관심 | Loss ↓ but never 0, 계속 개선 |

---

## Regularization

Loss만 최소화하면 overfitting이 생길 수 있습니다. Regularization term을 추가해 W가 너무 커지는 걸 방지합니다.

$$L = \underbrace{\frac{1}{N} \sum_i L_i}_{\text{data loss}} + \underbrace{\lambda R(W)}_{\text{regularization}}$$

- **목적**: Overfitting 방지 + 최적화 안정성 향상
- 대표적인 방법: L2 regularization (weight decay), L1, Dropout 등

<span style="color:#60a5fa">L2와 L1은 선호하는 가중치의 형태가 다릅니다. **L2**는 $\sum W^2$을 최소화하므로 가중치가 전체적으로 고르게 작아지는 것을 선호합니다(spread-out weights). **L1**은 $\sum |W|$을 최소화하므로 불필요한 가중치를 정확히 0으로 만드는 sparse한 해를 선호합니다. $\lambda$는 data loss와 regularization 사이의 균형을 조절하는 hyperparameter로, 클수록 모델이 단순해지고 작을수록 데이터에 더 fit하게 됩니다.</span>

---

## 핵심 질문

1. **KNN의 train/test time complexity는?** Training O(1), Inference O(N). 딥러닝과 정반대이므로 실제 적용이 어렵다.

2. **L1과 L2 distance의 차이는?** L1은 축 방향에 예민한 마름모형 경계, L2는 원형 경계를 만든다. 좌표계의 의미가 명확할 때는 L1, 전체적인 거리가 중요할 때는 L2가 유리하다.

3. **Multiclass SVM Loss에서 모든 score가 0에 가까울 때 예상 loss는?** C − 1 (클래스 수 − 1). 초기화 sanity check에 활용된다.

4. **Softmax에서 초기화 시 s ≈ 0이면 loss가 log(C)인 이유는?** 모든 클래스 확률이 1/C로 균등해지므로 $-\log(1/C) = \log(C)$.

5. **Bias term b의 역할은?** Decision boundary가 원점을 반드시 지나야 하는 제약을 제거한다. b가 없으면 표현력이 크게 제한된다.

6. **SVM Loss와 Softmax Loss의 근본적인 차이는?** SVM은 정답 score가 margin 이상이면 Loss = 0으로 무관심하지만, Softmax는 확률이 1이 되기 전까지 항상 개선하려 한다.

7. **Regularization의 목적 두 가지는?** ① Overfitting 방지, ② 동일한 data loss를 내는 W 중 더 단순한(일반화 잘 되는) 해를 선택.

8. **L2 vs L1 regularization의 차이는?** L2는 가중치를 전체적으로 균등하게 작게 만들고, L1은 불필요한 가중치를 0으로 만드는 sparse 해를 유도한다.
