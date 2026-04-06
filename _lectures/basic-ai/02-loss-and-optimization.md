---
title: "Loss Function & Optimization"
course: basic-ai
course_title: "Basic of AI"
order: 2
description: "모델이 얼마나 틀렸는지 측정하는 손실 함수와, 이를 최소화하는 경사하강법의 원리를 설명합니다."
mathjax: true
---

## 학습이란?

머신러닝에서 **학습(Training)**은 모델이 올바른 답을 내도록 파라미터(가중치)를 조정하는 과정입니다.

이 과정의 핵심:

1. 모델이 예측을 출력
2. 정답과 비교해서 **얼마나 틀렸는지** 계산 → **손실 함수**
3. 손실을 줄이는 방향으로 파라미터 업데이트 → **최적화**

---

## 손실 함수 (Loss Function)

손실 함수 $\mathcal{L}$은 모델의 예측값과 정답의 차이를 하나의 숫자로 표현합니다.

### MSE (Mean Squared Error) — 회귀
$$
\mathcal{L}_{\text{MSE}} = \frac{1}{N} \sum_{i=1}^{N} (y_i - \hat{y}_i)^2
$$

- $y_i$: 정답, $\hat{y}_i$: 예측값
- 큰 오차에 더 큰 패널티 (제곱이기 때문)

### Cross-Entropy Loss — 분류
$$
\mathcal{L}_{\text{CE}} = -\sum_{c=1}^{C} y_c \log(\hat{p}_c)
$$

- $\hat{p}_c$: 클래스 $c$에 대한 예측 확률
- 올바른 클래스의 확률이 낮을수록 손실 증가

---

## 경사하강법 (Gradient Descent)

손실을 최소화하려면 파라미터 $\theta$를 어느 방향으로 움직여야 할까요?

> **손실이 증가하는 방향의 반대** = 기울기(gradient)의 반대 방향

$$
\theta \leftarrow \theta - \eta \cdot \nabla_\theta \mathcal{L}
$$

- $\eta$: 학습률(learning rate) — 한 번에 얼마나 이동할지
- $\nabla_\theta \mathcal{L}$: 파라미터에 대한 손실의 기울기

---

## 학습률의 중요성

| 학습률 | 결과 |
|---|---|
| 너무 크다 | 최솟값을 지나쳐 발산할 수 있음 |
| 너무 작다 | 학습이 매우 느리고 local minimum에 갇힘 |
| 적절 | 안정적으로 수렴 |

---

## SGD vs. Mini-batch GD

| 방법 | 설명 | 특징 |
|---|---|---|
| **Full-batch GD** | 전체 데이터로 기울기 계산 | 정확하지만 느림 |
| **SGD** | 데이터 1개씩 업데이트 | 빠르지만 불안정 |
| **Mini-batch GD** | 작은 배치(32, 64, 128 등) 단위로 업데이트 | 균형점, 실제로 가장 많이 사용 |

---

## 다음 강의 예고

다음 시간에는 **신경망(Neural Network)**의 구조를 다룹니다. 퍼셉트론부터 다층 신경망까지, 어떻게 복잡한 함수를 근사하는지 살펴봅니다.
