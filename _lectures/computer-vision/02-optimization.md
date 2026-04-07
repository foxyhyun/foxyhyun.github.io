---
title: "Optimization"
course: computer-vision
course_title: "Computer Vision"
order: 2
description: "Gradient Descent부터 SGD, Momentum, RMSProp, Adam까지 딥러닝 최적화 알고리즘의 흐름을 정리합니다."
mathjax: true
---

## Gradient란?

**Gradient**는 각 파라미터를 조금 바꿨을 때 Loss가 얼마나 민감하게 변하는지를 모아둔 벡터입니다.

전체 Loss와 그 gradient는 다음과 같이 정의됩니다.

$$
L(W) = \frac{1}{N}\sum_{i=1}^N L_i(x_i, y_i, W) + \lambda R(W)
$$

$$
\nabla_W L(W) = \frac{1}{N}\sum_{i=1}^N \nabla_W L_i(x_i, y_i, W) + \lambda \nabla_W R(W)
$$

모든 항에 대해 gradient를 계산하면 계산량이 매우 많아집니다. 이를 해결하기 위한 것이 SGD입니다.

---

## SGD (Stochastic Gradient Descent)

전체 데이터 대신 **mini-batch** 단위로 gradient를 근사해 업데이트합니다.

$$
x_{t+1} = x_t - \alpha \nabla f(x_t)
$$

SGD의 한계:

- Loss surface가 한 방향으로 가파르고 다른 방향으로 완만할 때 **지그재그로 진동**하며 느리게 수렴

- Local minimum이나 saddle point 근처에서 gradient가 작아져 **학습 정체**

- Mini-batch로 gradient를 근사하므로 업데이트에 **noise** 포함

---

## Momentum

SGD는 매번 현재 gradient만 보고 움직입니다. Momentum은 **이전 업데이트 방향(velocity)**도 함께 반영해 진동을 줄이고 수렴을 가속합니다.

$$
v_{t+1} = \rho v_t - \alpha \nabla f(x_t)
$$

$$
x_{t+1} = x_t + v_{t+1}
$$

- 모든 파라미터에 동일한 learning rate를 적용

- Saddle point나 local minimum에서 더 빠르게 탈출 가능

---

## RMSProp

파라미터마다 **다른 learning rate**를 적용합니다.

- gradient가 자주 크게 발생하는 방향 → 분모가 커져 → update 작게

- gradient가 작은 방향 → 분모가 작아져 → update 크게

이를 통해 가파른 방향의 진동을 줄이고, 완만한 방향의 수렴을 가속합니다.

---

## Adam

**Momentum + RMSProp**을 결합한 알고리즘입니다.

- **방향**: Momentum으로 부드럽게

- **Step 크기**: RMSProp처럼 적응적으로 조절

권장 하이퍼파라미터:

| 파라미터 | 권장값 |
|---|---|
| Momentum ($\beta_1$) | 0.9 |
| RMSProp ($\beta_2$) | 0.999 |
| Learning Rate | 1e-3 또는 5e-4 |

최근에는 **AdamW** (Weight Decay 추가)가 더 많이 사용됩니다.

---

## 1차 vs 2차 최적화

| | 1차 최적화 | 2차 최적화 |
|---|---|---|
| 사용 정보 | Gradient | Gradient + Hessian |
| 근사 방식 | 선형 근사 | 이차 근사 (곡률 반영) |
| 계산 비용 | 낮음 | 매우 높음 (Hessian 역행렬) |
| 실용성 | 딥러닝에서 표준 | 대규모 모델에 부적합 |

대규모 딥러닝 모델에는 계산 효율이 훨씬 좋은 **1차 최적화**가 사실상 표준입니다.
