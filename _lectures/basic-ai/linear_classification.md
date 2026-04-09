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

이 수식에서 각 행렬의 크기가 왜 저렇게 되는지 이해하는 것이 중요합니다.

**입력 $x$: (3072×1)**
32×32 크기의 RGB 이미지는 32×32×3 = 3072개의 픽셀 값으로 이루어져 있습니다. 이 2D 이미지를 하나의 긴 열벡터로 펼친(flatten) 것이 $x$입니다.

**가중치 $W$: (10×3072)**
분류해야 할 클래스가 10개라면, 각 클래스마다 3072개의 픽셀에 대한 가중치가 필요합니다. 즉 W는 "클래스 수 × 입력 차원" 형태입니다. $W$의 각 행(row)이 하나의 클래스에 해당하는 가중치 벡터입니다.

**편향 $b$: (10×1)**
클래스마다 독립적인 오프셋 값 하나씩, 총 10개입니다.

**출력 $f(x,W)$: (10×1)**
행렬 곱 $(10\times3072) \cdot (3072\times1)$의 결과는 $(10\times1)$이 됩니다. 여기에 $b(10\times1)$를 더하면 **10개 클래스 각각에 대한 점수(score)**가 나옵니다.

<figure>
  <img src="{{ '/assets/lecture_img/linear_classification/img7.png' | relative_url }}" alt="행렬 연산 시각화">
  <figcaption>이미지를 1D 벡터로 펼친 뒤 W를 곱하고 b를 더하면 각 클래스의 점수가 출력됩니다. 예시에서는 Cat: 1.1−96.8, Dog: 3.2+437.9, Ship: −1.2+61.95 처럼 b가 점수를 보정합니다.</figcaption>
</figure>

- **W (weight matrix)**: 어떤 픽셀 패턴이 어떤 클래스에 해당하는지 학습한 정보
- **b (bias)**: 데이터와 무관하게 클래스마다 고유한 오프셋을 부여합니다. b가 없다면 decision boundary가 반드시 원점을 지나야 하는 제약이 생깁니다.

<span style="color:#60a5fa">W의 각 행(row)은 해당 클래스에 대한 **템플릿**으로 해석할 수 있습니다. 예를 들어 "자동차" 행은 자동차처럼 생긴 픽셀 패턴을 담고 있고, 입력 이미지와의 내적(dot product)이 클수록 해당 클래스와 유사하다고 판단합니다. 이런 관점에서 Linear Classification을 "학습된 **템플릿 매칭**"이라고도 부릅니다.</span>

---

## Loss Function

W가 얼마나 좋은지 어떻게 알 수 있을까요? 바로 **Loss Function**으로 측정합니다. Loss는 현재 W가 만들어낸 예측이 정답과 얼마나 다른지를 하나의 숫자로 표현합니다. Loss가 낮을수록 좋은 W입니다.

### Multiclass SVM Loss (Hinge Loss)

SVM Loss의 핵심 아이디어는 간단합니다. 정답 클래스의 점수가 오답 클래스들보다 **충분한 margin만큼 높아야** 한다는 것입니다.

$$L_i = \sum_{j \neq y_i} \max(0,\ s_j - s_{y_i} + 1)$$

$$L = \frac{1}{N} \sum_i L_i$$

이 수식을 그래프로 표현하면 아래와 같습니다.

<figure>
  <img src="{{ '/assets/lecture_img/linear_classification/img8.png' | relative_url }}" alt="Hinge Loss 그래프">
  <figcaption>x축은 (정답 score − 오답 score)의 차이, y축은 해당 오답 클래스에서 발생하는 loss입니다.</figcaption>
</figure>

그래프의 x축은 $s_{y_i} - s_j$, 즉 **정답 클래스 점수와 오답 클래스 점수의 차이**입니다. 이 차이가 클수록 정답이 오답보다 훨씬 높은 점수를 받고 있다는 뜻입니다.

- **기울기가 있는 구간** (왼쪽): 차이가 1보다 작을 때, 즉 오답 클래스 점수가 정답 클래스 점수보다 1 미만으로만 낮을 때입니다. 이 경우 $s_j - s_{y_i} + 1 \gt 0$이므로 loss가 발생하며, 차이가 줄어들수록(정답이 오답에 비해 덜 높을수록) loss가 선형으로 증가합니다. 기울기가 −1인 이유는 정답 score가 1 높아질수록 loss가 정확히 1씩 줄어들기 때문입니다.
- **loss = 0인 구간** (오른쪽): 차이가 1 이상일 때입니다. 정답이 오답보다 margin(1) 이상 높으면 이미 충분히 잘 분류된 것으로 보고 loss를 0으로 처리합니다. 이 지점을 **"경계(hinge)"**라고 부릅니다.

<span style="color:#60a5fa">이 그래프가 **Hinge**라는 이름이 붙은 이유는 꺾이는 형태가 문의 경첩(hinge)처럼 생겼기 때문입니다. 기울어진 부분과 수평 부분이 만나는 꺾이는 지점이 핵심입니다. SVM은 이 hinge 덕분에 정답이 충분히 높으면 gradient가 0이 되어 더 이상 그 샘플에서 학습 신호가 오지 않습니다. 즉, "이미 잘 분류된 샘플은 무시하고 어려운 샘플에 집중"하는 효과가 생깁니다.</span>

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

위 예시를 바탕으로 자주 나오는 질문들을 살펴봅니다.

**Q1: Car score가 0.5 감소하면 loss가 어떻게 될까요?**

위 예시에서 car가 정답인 이미지의 점수는 cat 3.2, car 5.1, frog −1.7입니다. car가 정답이므로 오답 클래스인 cat과 frog에 대해 loss를 계산합니다.

car score가 5.1 → 4.6으로 감소하면:
- cat에 대해: max(0, 3.2 − 4.6 + 1) = max(0, −0.4) = **0**
- frog에 대해: max(0, −1.7 − 4.6 + 1) = max(0, −5.3) = **0**

car score가 충분히 높아서 0.5 감소해도 여전히 margin을 만족합니다. **Loss는 변하지 않습니다.**

<span style="color:#60a5fa">이것이 SVM Loss의 핵심 성질입니다. 정답 score가 오답보다 margin(1) 이상 높은 "안전 구간"에 있는 한, 정답 score가 조금 바뀌어도 loss에 전혀 영향을 주지 않습니다. 이 구간에서는 **gradient도 0**이 되어 해당 샘플에서 파라미터 업데이트가 일어나지 않습니다.</span>

**Q2: SVM Loss의 최솟값과 최댓값은?**

- **최솟값 = 0**: 정답 클래스가 모든 오답 클래스보다 margin(1) 이상 높으면 $\max(0, \cdot)$의 모든 항이 0이 됩니다.
- **최댓값 = ∞**: 오답 클래스의 score가 정답보다 무한히 높아질 수 있으므로 loss도 무한히 커질 수 있습니다.

**Q3: 학습 초반 W가 작아 모든 score ≈ 0이면 loss는 얼마일까요?**

모든 $s \approx 0$이면 $s_j - s_{y_i} + 1 \approx 0 - 0 + 1 = 1$이 C−1개의 오답 클래스에서 발생합니다.

$$L_i \approx (C - 1) \times 1 = C - 1$$

3개 클래스라면 초기 loss ≈ 2, 10개 클래스라면 초기 loss ≈ 9입니다. **이 값을 이용해 구현이 올바른지 sanity check 할 수 있습니다.**

**Q4: 합산에 정답 클래스(j = y_i)도 포함하면 어떻게 될까요?**

원래 수식은 $j \neq y_i$로 정답 클래스를 제외합니다. 만약 정답 클래스도 포함하면 $j = y_i$일 때 $s_{y_i} - s_{y_i} + 1 = 1$이 항상 더해집니다. 즉, **모든 이미지의 loss가 정확히 1씩 증가**합니다. 최솟값이 0이 아니라 1이 되므로 수식의 의미가 달라집니다. 관례적으로 정답 클래스는 제외합니다.

**Q5: sum 대신 mean을 쓰면 어떻게 될까요?**

클래스 수 C로 나누는 것이므로 loss 값의 **스케일만 달라지고, 어떤 W가 더 좋은지의 순서(ranking)는 변하지 않습니다.** 최솟값은 여전히 0, 최댓값은 ∞입니다. 실제로 mean을 쓰는 구현도 있지만 결과에 질적인 차이는 없습니다.

**Q6: Squared Hinge Loss $\max(0, s_j - s_{y_i} + 1)^2$를 쓰면 어떻게 달라질까요?**

일반 Hinge Loss는 margin을 넘어서면 loss가 선형으로 증가하지만, Squared Hinge Loss는 제곱으로 증가합니다. 이 의미는 **조금 틀린 것은 괜찮지만, 많이 틀린 것에는 훨씬 강한 패널티**를 준다는 것입니다. 어떤 loss를 쓸지는 문제에 따라 다르며, 두 손실 함수는 서로 다른 W를 최적해로 만들 수 있습니다.

<span style="color:#60a5fa">Squared Hinge Loss는 outlier(극단적으로 틀린 예시)에 매우 민감합니다. 반면 일반 Hinge Loss는 틀린 정도에 비례해 선형으로 페널티를 줍니다. 데이터에 노이즈나 레이블 오류가 있다면 squared보다 일반 hinge가 더 robust할 수 있습니다.</span>

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

<span style="color:#60a5fa">초기화 시 loss가 log(C)인 이유를 직관적으로 이해해봅시다. s ≈ 0이면 exp(s) ≈ 1이므로 모든 클래스의 확률이 1/C로 균등해집니다. 정답 확률도 1/C이므로 $-\log(1/C) = \log(C)$가 됩니다. 예를 들어 10개 클래스라면 초기 loss는 log(10) ≈ 2.3이어야 합니다. 학습 시작 시 이 값 근처인지 확인하는 것은 구현이 올바른지 점검하는 **sanity check**입니다.<br><br>또한 수치 안정성을 위해 실제 구현에서는 exp 계산 전에 모든 score에서 최댓값을 빼줍니다.</span>

$$P(Y=k \mid X=x_i) = \frac{e^{s_k - \max_j s_j}}{\sum_j e^{s_j - \max_j s_j}}$$

<span style="color:#60a5fa">수학적으로 결과 확률은 동일하지만, 분자/분모에서 동일한 값을 나눠 exp의 입력이 항상 0 이하가 되어 **overflow를 방지**할 수 있습니다.</span>

### SVM vs Softmax

두 classifier의 가장 큰 차이는 **정답이 충분히 높을 때의 태도**입니다.

| | SVM | Softmax |
|---|---|---|
| 출력 해석 | 상대적 score | 확률 (0~1) |
| Loss | Hinge Loss | Cross-Entropy |
| 정답 score가 이미 충분히 높다면 | Loss = 0, 더 이상 신경 쓰지 않음 | Loss가 낮아지긴 하지만 절대 0이 되지 않음 |

SVM은 "그 정도면 충분해"라며 넘어가지만, Softmax는 "확률이 1이 될 때까지 계속 개선하려 한다"고 이해하면 됩니다.

---

## Regularization

훈련 데이터에서 loss를 낮추는 것만이 목표가 아닙니다. 새로운 데이터에서도 잘 동작해야 합니다.

### 왜 Regularization이 필요한가?

다음 상황을 생각해봅시다. 훈련 데이터의 loss를 완벽하게 0으로 만드는 W가 있다고 해도, 그 W가 **유일하지 않을 수 있습니다.** $\alpha W$($\alpha > 1$)도 모든 score의 순위를 유지하면서 SVM Loss를 0으로 만들 수 있습니다. 즉, W를 2배로 키워도 loss가 동일할 수 있습니다.

그렇다면 어느 W가 더 좋을까요? 단순히 loss만 보면 구분할 수 없습니다. 여기서 regularization이 개입합니다.

또 다른 이유로, data loss만 줄이다 보면 모델이 **훈련 데이터의 노이즈나 이상치까지 외워버리는 overfitting**이 생깁니다. 훈련 데이터에서는 완벽하지만 새로운 데이터에서는 엉망인 모델이 됩니다.

Regularization은 이 두 문제를 동시에 해결합니다.

$$L = \underbrace{\frac{1}{N} \sum_i L_i}_{\text{data loss}} + \underbrace{\lambda R(W)}_{\text{regularization}}$$

- **data loss**: 현재 W가 훈련 데이터를 얼마나 잘 맞추는지
- **regularization loss**: W가 얼마나 "단순한지" — 크고 복잡한 W에 페널티

$\lambda$는 두 항의 균형을 조절하는 hyperparameter입니다. 클수록 단순한 모델, 작을수록 데이터에 더 fit한 모델이 됩니다.

### Regularization의 두 가지 목적

1. **Overfitting 방지**: W가 훈련 데이터의 세부 노이즈까지 외우는 것을 억제합니다.
2. **모델 선택**: 동일한 data loss를 내는 W가 여럿 있을 때, 더 단순하고 일반화 잘 되는 W를 선택하게 유도합니다.

대표적인 Regularization 방식은 다음과 같습니다.

$$R(W) = \sum_k \sum_l W_{k,l}^2 \quad \text{(L2, Weight Decay)}$$

$$R(W) = \sum_k \sum_l |W_{k,l}| \quad \text{(L1)}$$

<span style="color:#60a5fa">**L2**와 **L1**은 선호하는 가중치의 형태가 다릅니다. L2는 모든 가중치를 전체적으로 균등하게 작게 만들고, L1은 불필요한 가중치를 정확히 0으로 만드는 **sparse**한 해를 선호합니다. L2 regularization은 "특정 픽셀 하나에 지나치게 의존하지 말고 전체적으로 고루 보라"는 의미로 해석할 수 있습니다. 반면 L1은 "불필요한 특징은 완전히 무시하라"는 방향입니다. $\lambda$가 클수록 regularization이 강해져 모델이 단순해지고, 너무 크면 underfitting이 생길 수 있습니다.</span>

---

## 핵심 질문

1. **KNN의 train/test time complexity는 각각 얼마이며, 실제 사용이 어려운 이유는?**
Training은 O(1), Inference는 O(N). 실제 서비스에서는 추론 속도가 빨라야 하는데 KNN은 정반대이므로 적합하지 않습니다.

2. **L1과 L2 distance의 차이, 그리고 각각 어떤 경우에 유리한가?**
L1은 축 방향으로 예민한 마름모형 경계, L2는 원형 경계를 만듭니다. 각 차원이 독립적인 의미를 가질 때는 L1, 전체적인 거리가 중요할 때는 L2가 유리합니다.

3. **f(x, W) = Wx + b에서 각 행렬의 크기가 어떻게 결정되는가?**
입력 이미지를 flatten한 차원을 D, 클래스 수를 C라 하면 $x$는 (D×1), $W$는 (C×D), $b$는 (C×1), 출력은 (C×1)입니다. 32×32×3 이미지와 10개 클래스라면 D=3072, C=10.

4. **SVM Loss에서 정답 score가 조금 감소해도 loss가 변하지 않는 이유는?**
정답 score가 오답보다 margin(1) 이상 높은 "안전 구간"에 있으면 $\max(0, \cdot) = 0$이 유지됩니다. 이 구간에서는 gradient도 0이라 파라미터 업데이트도 없습니다.

5. **학습 초반 모든 score ≈ 0일 때 SVM Loss와 Softmax Loss의 예상값은?**
SVM: C−1 (오답 클래스 수만큼 margin 1이 누적), Softmax: log(C) (모든 클래스 확률이 균등하게 1/C가 되므로).

6. **Regularization이 필요한 이유 두 가지는?**
① Data loss를 0으로 만드는 W가 여럿 존재할 수 있어 그 중 더 단순한(일반화 잘 되는) W를 선택하기 위해, ② 훈련 데이터에 overfit되는 것을 방지하기 위해.

7. **SVM Loss와 Softmax Loss의 근본적인 차이는?**
SVM은 정답 score가 margin을 충족하면 loss = 0으로 무관심합니다. Softmax는 정답 확률이 1이 되기 전까지 계속 개선하려 합니다.

8. **L2와 L1 regularization의 차이는?**
L2는 모든 가중치를 전체적으로 균등하게 작게 만들고(spread-out), L1은 불필요한 가중치를 0으로 만드는 sparse한 해를 선호합니다.

9. **Hinge Loss 그래프에서 기울기가 −1인 이유는?**
$L_i = s_j - s_{y_i} + 1$에서 정답 score $s_{y_i}$가 1 증가하면 loss가 정확히 1 감소하기 때문입니다. 반대로 오답 score $s_j$가 1 증가하면 loss가 1 증가합니다.

10. **Bias term b가 없으면 어떤 문제가 생기나?**
Decision boundary가 반드시 원점을 지나야 하는 제약이 생겨 표현력이 크게 제한됩니다.
