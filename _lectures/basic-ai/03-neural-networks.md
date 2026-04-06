---
title: "Neural Networks"
course: basic-ai
course_title: "Basic of AI"
order: 3
description: "퍼셉트론부터 다층 신경망까지, 뉴럴 네트워크가 어떻게 복잡한 함수를 학습하는지 설명합니다."
mathjax: true
---

## 퍼셉트론 (Perceptron)

뉴럴 네트워크의 최소 단위입니다.

$$
\hat{y} = f\!\left(\sum_{i} w_i x_i + b\right)
$$

- $x_i$: 입력
- $w_i$: 가중치(weight)
- $b$: 편향(bias)
- $f$: 활성화 함수(activation function)

---

## 활성화 함수

단순 선형 변환만 쌓으면 아무리 층을 추가해도 표현력이 증가하지 않습니다. **비선형 활성화 함수**가 필요한 이유입니다.

### Sigmoid
$$
\sigma(x) = \frac{1}{1 + e^{-x}} \in (0, 1)
$$
- 출력을 확률로 해석하기 좋음
- 단점: 깊은 네트워크에서 기울기 소실(Vanishing Gradient) 문제

### ReLU (Rectified Linear Unit)
$$
\text{ReLU}(x) = \max(0, x)
$$
- 현재 가장 많이 쓰이는 기본 활성화 함수
- 계산이 단순하고 깊은 네트워크에서도 학습이 잘 됨

### Softmax
$$
\text{softmax}(x_i) = \frac{e^{x_i}}{\sum_j e^{x_j}}
$$
- 출력을 **클래스별 확률 분포**로 변환
- 다중 클래스 분류의 마지막 레이어에 사용

---

## 다층 신경망 (MLP)

여러 퍼셉트론을 층(layer)으로 쌓은 구조입니다.

```
Input Layer → Hidden Layer(s) → Output Layer
   x₁, x₂        h₁, h₂, h₃        ŷ
```

- **Input Layer**: 입력 데이터
- **Hidden Layer**: 특징을 변환/추출
- **Output Layer**: 최종 예측

층이 깊을수록 더 복잡한 패턴을 표현할 수 있습니다.

---

## 역전파 (Backpropagation)

파라미터($w, b$)를 학습하는 알고리즘입니다. **연쇄 법칙(Chain Rule)**을 이용해 손실의 기울기를 출력층에서 입력층 방향으로 전파합니다.

$$
\frac{\partial \mathcal{L}}{\partial w_i} = \frac{\partial \mathcal{L}}{\partial \hat{y}} \cdot \frac{\partial \hat{y}}{\partial h} \cdot \frac{\partial h}{\partial w_i}
$$

- **Forward pass**: 입력 → 예측 계산
- **Backward pass**: 손실 → 기울기 역방향 전파
- 기울기를 이용해 경사하강법으로 파라미터 업데이트

---

## 요약

| 개념 | 역할 |
|---|---|
| 가중치 $w$ | 특징의 중요도 |
| 편향 $b$ | 활성화 임계값 조정 |
| 활성화 함수 | 비선형성 부여 |
| 역전파 | 자동 기울기 계산 |
| 경사하강법 | 파라미터 업데이트 |

---

## 다음 강의 예고

다음 시간에는 **과적합(Overfitting)과 정규화(Regularization)**를 다룹니다. 모델이 훈련 데이터에만 잘 맞고 새로운 데이터에 실패하는 이유와 해결책을 살펴봅니다.
