---
title: "Linear Classification"
course: basic-ai
course_title: "Basic of AI"
order: 1
description: "Image Classification의 한계를 넘어 Parametric Approach를 도입하고, SVM Loss와 Softmax로 학습 가능한 분류기를 만드는 방법을 다룹니다."
mathjax: true
---

## Image Classification의 어려움

이미지 분류는 단순해 보이지만 실제로는 상당히 까다로운 문제입니다. 인간은 고양이 사진을 보면 즉시 "고양이"라고 인식하지만, 컴퓨터에게 이미지란 그저 수백만 개의 숫자 배열일 뿐입니다. 같은 고양이를 찍더라도 조명이 바뀌거나 각도가 달라지면 픽셀 값은 완전히 달라집니다.

이 간극을 **Semantic Gap**이라고 합니다. 이미지 분류가 어려운 이유를 조금 더 구체적으로 살펴보면 다음과 같습니다.

- **Viewpoint variation** — 같은 물체도 보는 각도에 따라 픽셀이 완전히 달라집니다.
- **Illumination** — 밝은 곳과 어두운 곳, 역광 등 조명 조건에 따라 모델이 쉽게 흔들립니다.
- **Occlusion** — 물체의 일부가 가려지더라도 사람은 무엇인지 알지만, 픽셀 기반 접근은 어렵습니다.
- **Deformation** — 고양이가 구부리거나 뛰는 자세를 취해도 같은 클래스여야 합니다.
- **Intraclass variation** — "의자"라는 클래스 하나에도 수천 가지 생김새가 존재합니다.
- **Background clutter** — 배경이 복잡하면 물체와 구분이 쉽지 않습니다.

---

## 전통적 방법 vs. 머신러닝

과거에는 사람이 직접 규칙을 작성했습니다. "고양이는 수염이 있고, 귀가 뾰족하며…" 같은 조건을 코드로 표현하는 식입니다. 하지만 이런 방식은 새로운 물체가 등장할 때마다 규칙을 처음부터 다시 짜야 한다는 치명적인 단점이 있습니다.

머신러닝 방식은 이를 뒤집습니다. 규칙을 사람이 직접 만드는 것이 아니라, 데이터를 충분히 보여주면 컴퓨터가 스스로 패턴을 찾도록 합니다.

| 접근 방식 | 설명 |
|---|---|
| **Traditional** | 사람이 직접 feature와 규칙을 설계 |
| **ML** | 데이터 수집 → 분류기 학습 → 새 이미지에 적용 |

---

## K-Nearest Neighbor (KNN)

머신러닝 기반 분류기 중 가장 직관적인 방법입니다. 핵심 아이디어는 단순합니다. 훈련 이미지를 모두 기억해두고, 새 이미지가 들어오면 저장된 이미지들과 거리를 비교해 가장 가까운 K개의 이웃이 가진 레이블로 결정합니다.

거리 측정에는 두 가지 방법이 주로 사용됩니다.

$$d_1(I_1, I_2) = \sum_p |I_1^p - I_2^p| \quad \text{(L1 distance)}$$

$$d_2(I_1, I_2) = \sqrt{\sum_p (I_1^p - I_2^p)^2} \quad \text{(L2 distance)}$$

L1 distance는 각 픽셀 차이의 절댓값을 합산하고, L2는 제곱합의 제곱근을 사용합니다. 결과적으로 L1은 축 방향으로 날카로운 경계를, L2는 원형에 가까운 경계를 만들어냅니다.

<span style="color:#60a5fa">L1과 L2 중 어느 쪽이 더 나은가는 데이터의 성질에 따라 다릅니다. 특징 벡터의 각 차원이 독립적인 의미를 가질 때(예: 사람 키, 몸무게)는 L1이, 차원 간의 전체적인 거리가 중요할 때는 L2가 더 적합합니다. 이미지에서는 보통 L2를 더 많이 사용합니다.</span>

### 그런데 KNN은 실제로 쓰기 어렵습니다

KNN의 치명적인 문제는 **추론 속도**입니다. 학습 단계는 그냥 이미지를 저장하기만 하면 되니 O(1)로 매우 빠릅니다. 하지만 새 이미지가 들어올 때마다 저장된 이미지 전체와 거리를 비교해야 하므로 추론이 O(N)으로 느립니다.

실제 서비스 환경에서는 정반대여야 합니다. 학습은 오래 걸려도 되지만, 추론은 빨라야 합니다. 이 점에서 KNN은 실용적이지 않습니다.

<span style="color:#60a5fa">딥러닝 모델은 이 문제를 잘 해결합니다. 학습에는 많은 시간이 걸리지만, 일단 학습된 가중치(W)만 저장하면 추론은 행렬 연산 몇 번으로 끝납니다. 데이터 수(N)와 무관하게 거의 일정한 속도를 유지합니다.</span>

### Hyperparameter 설정

K 값이나 어떤 거리 함수를 쓸지는 데이터를 보면서 직접 시도해봐야 압니다. 이처럼 학습으로 결정되지 않고 사람이 직접 설정해야 하는 값들을 **hyperparameter**라고 합니다.

중요한 원칙은 **test set은 최후에 딱 한 번만 사용**해야 한다는 것입니다. test 성능을 반복해서 확인하다 보면 어느새 그 test set에 맞춰 hyperparameter가 튜닝되어 실제 성능이 부풀려집니다. 그래서 보통 Train / Val / Test 세 개로 나눠 val로 모델을 선택하고, 최종 평가만 test로 합니다.

데이터가 적을 때는 **Cross-validation**을 씁니다. 전체 데이터를 K개로 쪼개서 각각을 돌아가며 val로 사용하면 더 신뢰할 수 있는 평가가 가능합니다.

### Curse of Dimensionality

KNN에는 또 다른 근본 문제가 있습니다. 이미지 같은 고차원 데이터에서는 공간이 너무 넓어져 데이터가 완전히 희박해집니다. 64×64 이미지만 해도 4096차원인데, 이 공간을 의미 있게 채우려면 데이터 수가 지수적으로 늘어나야 합니다. 현실적으로 불가능한 양입니다.

---

## Linear Classification

### Parametric Approach

KNN의 문제를 해결하는 핵심 아이디어는 **파라미터를 학습**하는 것입니다. 이미지를 전부 기억하는 대신, 데이터로부터 가중치 행렬 W와 편향 b를 학습해두고 추론 시에는 이것만 사용합니다.

$$f(x, W) = Wx + b$$

<figure>
  <img src="{{ '/assets/lecture_img/linear_classification/img1.png' | relative_url }}" alt="Parametric Approach 다이어그램">
  <figcaption>32×32×3 이미지(3072차원)에 W(10×3072)를 곱하고 b(10×1)를 더해 10개의 클래스 점수를 출력합니다.</figcaption>
</figure>

- **W (weight matrix)**: 어떤 픽셀 패턴이 어떤 클래스에 해당하는지 학습한 정보
- **b (bias)**: 데이터와 무관하게 클래스마다 고유한 오프셋을 부여합니다. b가 없다면 decision boundary가 반드시 원점을 지나야 하는 제약이 생깁니다.

<span style="color:#60a5fa">W의 각 행(row)은 해당 클래스에 대한 **템플릿**으로 해석할 수 있습니다. 예를 들어 "자동차" 행은 자동차처럼 생긴 픽셀 패턴을 담고 있고, 입력 이미지와의 내적(dot product)이 클수록 해당 클래스와 유사하다고 판단합니다. 이런 관점에서 Linear Classification을 "학습된 템플릿 매칭"이라고도 부릅니다.</span>

---

## Loss Function

W가 얼마나 좋은지 어떻게 알 수 있을까요? 바로 **Loss Function**으로 측정합니다. Loss는 현재 W가 만들어낸 예측이 정답과 얼마나 다른지를 하나의 숫자로 표현합니다. Loss가 낮을수록 좋은 W입니다.

### Multiclass SVM Loss (Hinge Loss)

SVM Loss의 핵심 아이디어는 간단합니다. 정답 클래스의 점수가 오답 클래스들보다 **충분한 margin만큼 높아야** 한다는 것입니다.

$$L_i = \sum_{j \neq y_i} \max(0,\ s_j - s_{y_i} + 1)$$

$$L = \frac{1}{N} \sum_i L_i$$

예를 들어 cat, car, frog 세 클래스를 분류한다고 할 때, 각 이미지에 대한 loss를 계산해보면 다음과 같습니다.

<figure>
  <img src="{{ '/assets/lecture_img/linear_classification/img2.png' | relative_url }}" alt="SVM Loss cat 예시">
  <figcaption>Cat이 정답일 때: max(0, 5.1−3.2+1) + max(0, −1.7−3.2+1) = 2.9 + 0 = 2.9</figcaption>
</figure>

<figure>
  <img src="{{ '/assets/lecture_img/linear_classification/img3.png' | relative_url }}" alt="SVM Loss frog 예시">
  <figcaption>Frog가 정답이고 정답 score가 이미 충분히 높다면 Loss = 0이 됩니다.</figcaption>
</figure>

<figure>
  <img src="{{ '/assets/lecture_img/linear_classification/img4.png' | relative_url }}" alt="SVM Loss car 예시">
  <figcaption>Car가 정답일 때의 loss 계산 과정</figcaption>
</figure>

<figure>
  <img src="{{ '/assets/lecture_img/linear_classification/img5.png' | relative_url }}" alt="SVM Loss 전체 평균">
  <figcaption>세 이미지에 대한 평균 Loss: L = (2.9 + 0 + 12.9) / 3 = 5.27</figcaption>
</figure>

자주 나오는 질문들을 정리하면 다음과 같습니다.

| 질문 | 답 |
|---|---|
| 정답 클래스 score가 조금 감소하면 loss가 올라가나요? | 정답 score가 margin 안에서 움직이는 한 loss는 변하지 않습니다. |
| Loss의 최솟값과 최댓값은? | 최솟값 0, 최댓값 ∞ |
| 학습 초반 W가 너무 작아 모든 score ≈ 0이라면? | Loss ≈ C − 1 (클래스 수 − 1) |
| 정답 클래스 j = y_i를 합산에 포함하면? | 각 이미지의 loss가 1씩 증가합니다. |

<span style="color:#60a5fa">margin을 1로 설정하는 것은 사실 임의적입니다. W의 스케일을 바꾸면 margin도 같이 바뀌기 때문에, 중요한 것은 1이라는 숫자 자체가 아니라 "정답이 오답보다 얼마나 더 높아야 하는가"라는 상대적인 여유입니다. 한편 $\max(0, \cdot)^2$를 사용하는 **Squared Hinge Loss**는 조금 틀린 것보다 많이 틀린 예시에 훨씬 강한 패널티를 줍니다. 어떤 loss를 쓸지는 문제에 따라 다릅니다.</span>

---

## Softmax Classifier

SVM은 점수들의 상대적 크기만 신경 씁니다. 정답이 오답보다 margin만큼 높으면 loss = 0으로 그냥 넘어갑니다. 반면 **Softmax**는 점수를 확률로 바꿔서 해석합니다. 정답 클래스의 확률이 얼마나 높은지를 직접 최적화하는 셈입니다.

$$p_j = \frac{e^{s_j}}{\sum_k e^{s_k}}$$

$$L_i = -\log\left(\frac{e^{s_{y_i}}}{\sum_k e^{s_k}}\right)$$

왜 negative log를 취할까요? 정답 클래스의 확률이 1에 가까울수록 loss가 0에 가까워지고, 확률이 낮을수록 loss가 크게 증가하길 원하기 때문입니다. $-\log$는 이 성질을 잘 표현합니다.

<figure>
  <img src="{{ '/assets/lecture_img/linear_classification/img6.png' | relative_url }}" alt="Softmax 확률 변환 과정">
  <figcaption>raw score → exp → normalize하면 합이 1인 확률 분포가 됩니다. 정답(cat)의 확률이 1에 가까울수록 Loss → 0.</figcaption>
</figure>

| 질문 | 답 |
|---|---|
| Loss의 최솟값/최댓값은? | 최솟값 0 (확률이 1일 때), 최댓값 ∞ |
| 초기화 시 모든 s ≈ 0이면 예상 loss는? | log(C) |

<span style="color:#60a5fa">초기화 시 loss가 log(C)인 이유를 직관적으로 이해해봅시다. s ≈ 0이면 exp(s) ≈ 1이므로 모든 클래스의 확률이 1/C로 균등해집니다. 정답 확률도 1/C이므로 $-\log(1/C) = \log(C)$가 됩니다. 예를 들어 10개 클래스라면 초기 loss는 log(10) ≈ 2.3이어야 합니다. 학습을 시작했을 때 loss가 이 값 근처인지 확인하는 것은 구현이 올바른지 점검하는 유용한 sanity check입니다.<br><br>또한 수치 안정성을 위해 실제 구현에서는 exp 계산 전에 모든 score에서 최댓값을 빼줍니다. $e^{s_j - \max s}$처럼요. 결과 확률은 동일하지만, 그대로 exp를 취하면 큰 수가 들어왔을 때 overflow가 발생할 수 있기 때문입니다.</span>

### SVM vs. Softmax

두 classifier의 가장 큰 차이는 **정답이 충분히 높을 때의 태도**입니다.

| | SVM | Softmax |
|---|---|---|
| 출력 해석 | 상대적 score | 확률 (0~1) |
| Loss | Hinge Loss | Cross-Entropy |
| 정답 score가 이미 충분히 높다면 | Loss = 0, 더 이상 신경 쓰지 않음 | Loss가 낮아지긴 하지만 절대 0이 되지 않음 |

SVM은 "그 정도면 충분해"라며 넘어가지만, Softmax는 "확률이 1이 될 때까지 계속 개선하려 한다"고 이해하면 됩니다.

---

## Regularization

훈련 데이터에서 loss를 낮추는 것만이 목표가 아닙니다. 새로운 데이터에서도 잘 동작해야 합니다. 훈련 데이터에만 지나치게 맞춰진 모델을 **overfit**됐다고 합니다.

이를 방지하기 위해 loss에 regularization term을 추가합니다.

$$L = \underbrace{\frac{1}{N} \sum_i L_i}_{\text{data loss}} + \underbrace{\lambda R(W)}_{\text{regularization}}$$

data loss만 최소화하면 W가 훈련 데이터의 노이즈까지 외워버릴 수 있습니다. Regularization은 W가 필요 이상으로 커지지 않도록 페널티를 줘서, 더 단순하고 일반화 잘 되는 모델을 유도합니다.

<span style="color:#60a5fa">L2와 L1은 선호하는 가중치의 형태가 다릅니다. **L2** ($\sum W^2$)는 모든 가중치를 전체적으로 균등하게 작게 만들고, **L1** ($\sum |W|$)은 불필요한 가중치를 정확히 0으로 만드는 sparse한 해를 선호합니다. $\lambda$가 클수록 regularization이 강해져 모델이 단순해지고, 너무 크면 underfitting이 생길 수 있습니다. 이 균형을 맞추는 것이 hyperparameter 튜닝의 핵심 중 하나입니다.</span>

---

## 핵심 질문

1. **KNN의 train/test time complexity는 각각 얼마이며, 실제 사용이 어려운 이유는?**
Training은 O(1), Inference는 O(N). 실제 서비스에서는 추론 속도가 빨라야 하는데 KNN은 정반대이므로 적합하지 않습니다.

2. **L1과 L2 distance의 차이, 그리고 각각 어떤 경우에 유리한가?**
L1은 축 방향으로 예민한 마름모형 경계, L2는 원형 경계를 만듭니다. 각 차원이 독립적인 의미를 가질 때는 L1, 전체적인 거리가 중요할 때는 L2가 유리합니다.

3. **Multiclass SVM Loss에서 학습 초반 모든 score ≈ 0이면 예상 loss는?**
C − 1. 정답을 제외한 오답 클래스 수만큼 margin 1이 누적되기 때문입니다.

4. **Softmax에서 학습 초반 모든 s ≈ 0이면 loss가 log(C)인 이유는?**
모든 클래스 확률이 1/C로 균등해지므로, 정답 확률도 1/C이고 $-\log(1/C) = \log(C)$가 됩니다.

5. **Bias term b가 없으면 어떤 문제가 생기나?**
Decision boundary가 반드시 원점을 지나야 하는 제약이 생겨 표현력이 크게 제한됩니다.

6. **SVM Loss와 Softmax Loss의 근본적인 차이는?**
SVM은 정답 score가 margin을 충족하면 loss = 0으로 무관심합니다. Softmax는 정답 확률이 1이 되기 전까지 계속 개선하려 합니다.

7. **Regularization의 목적 두 가지는?**
① Overfitting 방지, ② 동일한 data loss를 내는 여러 W 중 더 단순하고 일반화 잘 되는 해를 선택하게 유도.

8. **L2와 L1 regularization의 차이는?**
L2는 모든 가중치를 전체적으로 균등하게 작게 만들고(spread-out), L1은 불필요한 가중치를 0으로 만드는 sparse한 해를 선호합니다.
