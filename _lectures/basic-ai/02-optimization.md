---
title: "Optimization"
course: basic-ai
course_title: "Basic of AI"
order: 2
description: "Gradient Descent부터 SGD, Momentum, RMSProp, Adam까지 딥러닝 최적화 알고리즘의 흐름을 정리합니다."
mathjax: true
---

## Gradient란?

**Gradient**는 각 파라미터를 조금 바꿨을 때 Loss가 얼마나 민감하게 변하는지를 모아둔 벡터입니다. Loss를 낮추려면 gradient의 반대 방향으로 파라미터를 움직이면 됩니다. 이것이 최적화의 핵심 아이디어입니다.

전체 Loss와 그 gradient는 다음과 같이 정의됩니다.

$$L(W) = \frac{1}{N}\sum_{i=1}^N L_i(x_i, y_i, W) + \lambda R(W)$$

$$\nabla_W L(W) = \frac{1}{N}\sum_{i=1}^N \nabla_W L_i(x_i, y_i, W) + \lambda \nabla_W R(W)$$

모든 데이터에 대해 gradient를 계산하면 계산량이 매우 많아집니다. 이를 해결하기 위한 것이 SGD입니다.

<span style="color:#60a5fa">Gradient는 방향과 크기를 모두 가집니다. 크기가 클수록 그 파라미터가 Loss에 미치는 영향이 크다는 의미입니다. 실제로 gradient를 계산하는 방법은 두 가지입니다. **Numerical gradient**는 파라미터를 아주 조금 바꿔보며 Loss 변화를 직접 측정하는 방식으로, 구현이 쉽지만 느리고 근사값입니다. **Analytic gradient**는 수식을 미분해 정확하게 계산하는 방식으로, 빠르고 정확하지만 구현이 복잡합니다. 실제 딥러닝에서는 analytic gradient를 쓰고, 구현이 올바른지 확인할 때만 numerical gradient와 비교합니다(gradient check).</span>

---

## SGD (Stochastic Gradient Descent)

전체 데이터 대신 **mini-batch** 단위로 gradient를 근사해 업데이트합니다.

$$x_{t+1} = x_t - \alpha \nabla f(x_t)$$

직관적으로는 "Loss 표면에서 가장 가파르게 내려가는 방향으로 조금씩 이동한다"고 이해할 수 있습니다. 하지만 SGD에는 몇 가지 한계가 있습니다.

- Loss surface가 한 방향으로 가파르고 다른 방향으로 완만할 때 **지그재그로 진동**하며 느리게 수렴합니다.
- Local minimum이나 saddle point 근처에서 gradient가 0에 가까워져 **학습이 정체**됩니다.
- Mini-batch로 gradient를 근사하므로 업데이트마다 **noise**가 포함됩니다.

<span style="color:#60a5fa">딥러닝에서 **saddle point**는 생각보다 훨씬 흔합니다. 파라미터 수가 수백만 개인 고차원 공간에서는 어떤 방향으로는 Loss가 올라가고 다른 방향으로는 내려가는 지점이 매우 많습니다. SGD는 이런 지점에서 gradient ≈ 0이 되어 오랫동안 멈춰 있을 수 있습니다. **Local minimum**보다 saddle point가 실제로 더 문제가 되는 경우가 많습니다.</span>

---

## Momentum

SGD는 매번 현재 gradient만 보고 움직입니다. Momentum은 **이전 업데이트 방향(velocity)**도 함께 반영해 진동을 줄이고 수렴을 가속합니다. 공이 언덕을 굴러 내려오듯, 한번 방향이 잡히면 그 방향으로 계속 가속되는 효과입니다.

$$v_{t+1} = \rho v_t - \alpha \nabla f(x_t)$$

$$x_{t+1} = x_t + v_{t+1}$$

- $\rho$는 보통 0.9 또는 0.99를 사용합니다. 이전 velocity를 얼마나 유지할지 결정합니다.
- 모든 파라미터에 동일한 learning rate를 적용합니다.
- Saddle point나 local minimum 근처에서도 이전 velocity 덕분에 더 빠르게 탈출할 수 있습니다.

<span style="color:#60a5fa">Momentum의 변형으로 **Nesterov Momentum**이 있습니다. 일반 Momentum은 현재 위치에서 gradient를 계산한 뒤 velocity를 반영하는 반면, Nesterov는 velocity만큼 먼저 이동한 지점에서 gradient를 계산합니다. 이렇게 하면 "앞을 미리 보고" 방향을 조정하는 효과가 생겨 수렴이 더 안정적인 경우가 많습니다.</span>

---

## RMSProp

Momentum이 "방향"을 개선했다면, RMSProp은 **step 크기**를 파라미터마다 다르게 조절합니다.

$$s_{t+1} = \beta s_t + (1-\beta)(\nabla f(x_t))^2$$

$$x_{t+1} = x_t - \frac{\alpha}{\sqrt{s_{t+1}} + \epsilon} \nabla f(x_t)$$

- gradient가 자주 크게 발생하는 방향 → $s$가 커져 → 분모가 커져 → update **작게**
- gradient가 작은 방향 → $s$가 작아져 → 분모가 작아져 → update **크게**

이를 통해 가파른 방향의 진동을 줄이고, 완만한 방향의 수렴을 가속합니다.

<span style="color:#60a5fa">RMSProp의 아이디어는 **AdaGrad**에서 비롯됩니다. AdaGrad는 지금까지의 gradient 제곱합을 누적해 learning rate를 조절하는데, 학습이 길어질수록 분모가 계속 커져 learning rate가 0에 수렴해버리는 문제가 있습니다. RMSProp은 **지수 이동 평균(EMA)**으로 과거 gradient의 영향을 감쇠시켜 이 문제를 해결합니다. $\epsilon$은 분모가 0이 되는 것을 방지하기 위한 매우 작은 값(보통 1e-8)입니다.</span>

---

## Adam

**Momentum + RMSProp**을 결합한 알고리즘입니다. 방향은 Momentum으로, step 크기는 RMSProp처럼 적응적으로 조절합니다. 현재 딥러닝에서 가장 널리 사용되는 optimizer입니다.

$$m_{t+1} = \beta_1 m_t + (1-\beta_1)\nabla f(x_t) \quad \text{(1차 모멘트, 방향)}$$

$$v_{t+1} = \beta_2 v_t + (1-\beta_2)(\nabla f(x_t))^2 \quad \text{(2차 모멘트, 크기)}$$

$$x_{t+1} = x_t - \frac{\alpha \hat{m}_{t+1}}{\sqrt{\hat{v}_{t+1}} + \epsilon}$$

권장 하이퍼파라미터:

| 파라미터 | 권장값 |
|---|---|
| Momentum ($\beta_1$) | 0.9 |
| RMSProp ($\beta_2$) | 0.999 |
| Learning Rate | 1e-3 또는 5e-4 |

최근에는 **AdamW** (Weight Decay를 gradient가 아닌 파라미터에 직접 적용)가 더 많이 사용됩니다.

<span style="color:#60a5fa">수식에서 $\hat{m}$과 $\hat{v}$는 **bias correction** 항입니다. 학습 초반에는 $m$과 $v$가 0으로 초기화되어 있어 실제 gradient보다 작게 추정되는 문제가 생깁니다. 이를 $\hat{m} = m / (1-\beta_1^t)$로 보정해 초반 업데이트를 안정화합니다. **AdamW**와 일반 Adam의 차이는 weight decay의 적용 방식에 있습니다. 일반 Adam은 L2 regularization을 gradient에 추가하지만, AdamW는 파라미터에 직접 decay를 적용해 adaptive learning rate의 영향을 받지 않도록 합니다. Transformer 계열 모델에서는 AdamW가 표준입니다.</span>

---

## 1차 vs 2차 최적화

지금까지 살펴본 방법들은 모두 **1차 최적화**입니다. gradient(1차 미분)만 사용해 Loss surface를 선형으로 근사합니다. 반면 **2차 최적화**는 Hessian(2차 미분)을 추가로 활용해 곡률까지 반영합니다.

| | 1차 최적화 | 2차 최적화 |
|---|---|---|
| 사용 정보 | Gradient | Gradient + Hessian |
| 근사 방식 | 선형 근사 | 이차 근사 (곡률 반영) |
| 계산 비용 | 낮음 | 매우 높음 (Hessian 역행렬) |
| 실용성 | 딥러닝에서 표준 | 대규모 모델에 부적합 |

대규모 딥러닝 모델에는 계산 효율이 훨씬 좋은 **1차 최적화**가 사실상 표준입니다.

<span style="color:#60a5fa">2차 최적화가 이론적으로는 더 정확하지만 실용적이지 않은 이유는 **Hessian** 행렬의 크기 때문입니다. 파라미터가 N개라면 Hessian은 N×N 행렬입니다. 현대 딥러닝 모델의 파라미터가 수억 개라면 저장조차 불가능합니다. 이를 해결하려는 시도로 Hessian을 근사하는 **L-BFGS** 같은 방법이 있지만, 여전히 mini-batch 환경에서는 불안정하고 대규모 모델에는 맞지 않습니다.</span>

---

## 핵심 질문

1. **SGD의 세 가지 한계는?**
① 가파른/완만한 방향 차이로 인한 지그재그 진동, ② saddle point/local minimum에서 gradient ≈ 0으로 학습 정체, ③ mini-batch 근사로 인한 업데이트 noise.

2. **Momentum이 SGD의 어떤 문제를 해결하는가?**
이전 velocity를 유지해 진동을 줄이고, saddle point에서도 관성으로 빠져나올 수 있게 합니다. 하지만 모든 파라미터에 동일한 learning rate를 적용한다는 한계는 여전히 존재합니다.

3. **RMSProp의 핵심 아이디어는?**
파라미터마다 adaptive learning rate를 적용합니다. gradient가 큰 방향은 step을 작게, 작은 방향은 step을 크게 조절해 가파른 방향의 진동을 줄이고 완만한 방향의 수렴을 가속합니다.

4. **Adam이 Momentum, RMSProp과 다른 점은?**
두 방법을 결합한 것입니다. 1차 모멘트(방향)는 Momentum처럼, 2차 모멘트(크기)는 RMSProp처럼 관리합니다. 여기에 bias correction을 추가해 학습 초반의 불안정함도 해결합니다.

5. **Adam에서 bias correction이 필요한 이유는?**
$m$과 $v$가 0으로 초기화되어 학습 초반에 실제 값보다 작게 추정되기 때문입니다. $1/(1-\beta^t)$로 보정해 초반 업데이트를 안정화합니다.

6. **AdamW가 Adam과 다른 점은?**
Adam은 L2 regularization을 gradient에 더해 적용하는데, adaptive learning rate의 영향을 받아 weight decay 효과가 불균일해집니다. AdamW는 파라미터에 decay를 직접 적용해 이 문제를 해결합니다.

7. **2차 최적화가 딥러닝에서 사용되지 않는 이유는?**
파라미터 수가 N개일 때 Hessian 행렬이 N×N 크기라 수억 개 파라미터를 가진 현대 모델에서는 저장 자체가 불가능합니다.

8. **Numerical gradient와 Analytic gradient의 차이, 실제로 어느 쪽을 쓰는가?**
Numerical gradient는 파라미터를 조금 바꿔 Loss 변화를 직접 측정하는 근사값, Analytic gradient는 수식을 미분해 정확히 계산합니다. 실제 학습에는 빠른 Analytic gradient를 쓰고, 구현 검증에만 Numerical gradient로 비교(gradient check)합니다.
