---
title: "Neural Network"
course: basic-ai
course_title: "Basic of AI"
order: 3
description: "Feature Transform부터 Activation Function, Backpropagation, Universal Approximation, Normalization까지 신경망의 기초를 정리합니다."
mathjax: true
---

## Image Features

원래 공간(Original Space)에서는 선형 분류기(Linear Classifier)만으로 데이터를 구분하기 어려운 경우가 많다. 이때 **Feature Transform**을 적용하면, 복잡한 데이터를 선형으로 분리 가능한 공간으로 변환할 수 있다.

![img1](/assets/lecture_img/neural_network/img1.png)

대표적인 feature 표현 방식으로는 **HoG(Histogram of Oriented Gradients)** 가 있다. 이미지를 8×8 픽셀 단위로 나눈 뒤, 각 영역에서 edge 방향의 히스토그램을 계산해 texture와 윤곽 정보를 수치화한다.

![img2](/assets/lecture_img/neural_network/img2.png)

또 다른 방법으로는 **Bag of Visual Words** 가 있다. 이미지에서 눈, 발 같은 부분 특징들을 추출하고 이를 클러스터링해 "codebook"을 만든다. 이후 새로운 이미지를 각 클러스터 기준으로 인코딩하여 feature vector로 표현한다.

![img3](/assets/lecture_img/neural_network/img3.png)

<span style="color:#60a5fa">HoG와 Bag of Visual Words는 딥러닝 이전 시대의 대표적인 **hand-crafted feature** 방식입니다. 사람이 직접 "이미지에서 어떤 정보가 중요한지"를 정의해 추출했기 때문에 도메인 지식에 크게 의존합니다. 반면 현대 CNN은 데이터로부터 feature 자체를 학습하므로, 이러한 수작업 feature 설계를 대체했습니다. 그럼에도 불구하고 이 방식들을 이해하는 것은 "왜 비선형 변환이 필요한가"에 대한 직관을 얻는 데 중요합니다.</span>

---

## Activation Function

선형 분류기는 표현력에 한계가 있다. 이를 극복하기 위해 **hidden layer**와 **activation function**을 도입한다.

![img4](/assets/lecture_img/neural_network/img4.png)

신경망의 구성 요소를 정리하면 다음과 같다.

- **Depth**: 레이어의 수
- **Width**: 각 레이어의 크기 (유닛 수)
- **Parameters**: 학습을 통해 갱신되는 값. Weight와 Bias가 해당된다.
- **Activation**: 입력 샘플이 각 레이어를 통과할 때 중간에 계산되는 값.

신경망이 강력한 핵심 이유는 단순히 층을 많이 쌓아서가 아니다. **각 층 사이에 비선형성(nonlinearity)** 이 들어가기 때문이다. Activation function 없이 선형 연산만 쌓으면, 아무리 깊어도 결국 하나의 선형 함수와 동일해진다.

![img5](/assets/lecture_img/neural_network/img5.png)

대표적인 activation function으로는 Sigmoid, Tanh, **ReLU**, Leaky ReLU 등이 있으며, 현재는 ReLU와 그 변형들이 가장 널리 쓰인다.

<span style="color:#60a5fa">Sigmoid와 Tanh는 입력이 크거나 작을 때 gradient가 거의 0에 가까워지는 **vanishing gradient** 문제가 있습니다. 깊은 네트워크에서 backpropagation을 할 때 gradient가 층을 거치면서 점점 작아져 학습이 거의 불가능해지죠. ReLU는 양수 영역에서 gradient가 항상 1이기 때문에 이 문제를 크게 완화하고, 계산도 매우 단순합니다. 다만 음수 영역에서는 gradient가 0이 되어 뉴런이 영원히 비활성화되는 **dying ReLU** 문제가 있어, 이를 보완하기 위해 Leaky ReLU, PReLU, ELU, GELU 같은 변형들이 등장했습니다.</span>

---

## Gradient Descent

학습의 목표는 파라미터 θ를 조정해 **loss를 최소화**하는 것이다.

$$
\theta^* = \arg\min_\theta \sum_{i=1}^N L(f_\theta(x^{(i)}), y^{(i)})
$$

이를 위해 **Gradient Descent**를 사용한다. 현재 파라미터에서 gradient를 계산하고, 그 반대 방향으로 learning rate η만큼 이동한다.

$$
\theta^{k+1} \leftarrow \theta^k - \eta \nabla_\theta J(\theta^k)
$$

![img6](/assets/lecture_img/neural_network/img6.png)

각 파라미터에 대한 gradient는 **chain rule** 기반의 **backpropagation**으로 효율적으로 계산한다.

<span style="color:#60a5fa">Backpropagation의 핵심은 chain rule을 이용해 **출력단의 loss로부터 각 층의 gradient를 역방향으로 전파**하는 것입니다. 중간 계산 결과(activation)를 저장해두었다가 역전파 과정에서 재사용하기 때문에, 파라미터가 수백만 개여도 효율적으로 gradient를 구할 수 있습니다. 이 알고리즘이 없었다면 현대 딥러닝은 불가능했을 것입니다.</span>

---

## Neural Network 구조

기본적인 신경망은 다음과 같은 weight 구조를 가진다.

- **w1**: 입력층 → 은닉층 가중치
- **w2**: 은닉층 → 출력층 가중치

입력이 w1을 통해 은닉층으로 전달되고, activation function을 거친 뒤 w2를 통해 최종 출력이 만들어진다.

![img7](/assets/lecture_img/neural_network/img7.png)

---

## Feature Transform과 ReLU

Feature Transform은 **데이터의 표현 방식 자체를 바꾸는 것**이다.

![img8](/assets/lecture_img/neural_network/img8.png)

ReLU를 activation으로 사용하면 점들이 축 방향으로 몰리면서, 새로운 feature space에서 선형 분리가 가능해진다.

- Feature space에서 직선으로 나누는 것 = Original space에서 비선형 경계로 나누는 것

즉, **ReLU는 공간을 비선형적으로 접어서(fold), 선형 분류기가 더 강력하게 작동하도록 만든다.**

![img9](/assets/lecture_img/neural_network/img9.png)

전체 흐름을 정리하면 다음과 같다.

1. 원래 공간에서는 점들을 구분할 수 없다.
2. Feature transform으로 점들의 좌표를 새로운 공간으로 재배치한다.
3. ReLU를 적용하면 여러 위치에 흩어졌던 점들이 축 위나 원점 근처로 모인다.
4. 복잡하게 퍼져 있던 점들이 정리된 구조로 재배치되어 분리가 쉬워진다.

따라서 선형 transform만으로는 충분하지 않을 수 있고, ReLU 같은 **비선형 함수가 반드시 필요**하다.

유닛 수가 늘어날수록 network는 더 복잡한 패턴을 표현할 수 있다. 단, 모델 크기를 줄이는 방식으로 regularization을 하기보다는, **충분한 크기의 모델을 두고 L2 regularization을 강하게 적용하는 것**이 더 좋은 전략이다.

![img10](/assets/lecture_img/neural_network/img10.png)

<span style="color:#60a5fa">"큰 모델 + 강한 regularization"이 "작은 모델 + 약한 regularization"보다 일반적으로 더 좋은 성능을 보인다는 것은 딥러닝의 중요한 경험칙입니다. 작은 모델은 애초에 표현력이 부족해 복잡한 패턴을 배우지 못하지만, 큰 모델은 regularization으로 과적합만 적절히 막으면 훨씬 풍부한 표현을 학습할 수 있습니다. 이 원리는 현대 대규모 모델의 scaling law와도 연결됩니다.</span>

---

## Universal Approximation

ReLU network의 출력은 여러 개의 scaled ReLU를 합산한 것으로 볼 수 있다. ReLU 하나는 한 번 꺾이는 선형 함수다. 이를 여러 개 조합하면 **봉우리 모양(Bump Function)** 을 만들 수 있고, bump를 여러 개 쌓으면 임의의 복잡한 함수를 근사할 수 있다.

![img11](/assets/lecture_img/neural_network/img11.png)

이것이 **Universal Approximation Theorem**의 핵심이다. 충분한 수의 뉴런을 가진 단층 신경망은 임의의 연속 함수를 원하는 정밀도로 근사할 수 있다.

단, 이는 **표현 가능성(expressiveness)** 에 대한 이야기다. "이 지도 어딘가에 목적지가 존재한다"는 뜻이지, 실제로 gradient descent로 **찾을 수 있는지(학습 가능성)** 나 **일반화 성능**을 보장하지는 않는다.

<span style="color:#60a5fa">Universal Approximation Theorem은 "이론적으로는 단층 신경망만으로도 모든 연속 함수를 표현할 수 있다"는 강력한 결과지만, 실제로는 **깊은(deep)** 신경망이 훨씬 효율적입니다. 같은 함수를 근사하는 데 단층은 지수적으로 많은 뉴런이 필요한 반면, 깊은 네트워크는 층을 쌓는 것만으로 훨씬 적은 파라미터로 표현할 수 있습니다. 이것이 "deep learning"이 "wide learning"보다 선호되는 이유 중 하나입니다.</span>

---

## Convex vs Non-convex Optimization

Convex function은 최적화가 쉽고, global minimum으로의 수렴을 이론적으로 보장할 수 있다.

![img12](/assets/lecture_img/neural_network/img12.png)

반면 대부분의 신경망은 **non-convex optimization** 문제를 다룬다. 이론적으로 강한 수렴 보장을 주기 어렵지만, 실제로는 잘 작동하는 것이 알려져 있다.

<span style="color:#60a5fa">신경망의 loss surface가 non-convex임에도 gradient descent가 잘 작동하는 이유는 최근 연구에서 조금씩 밝혀지고 있습니다. 고차원 공간에서는 **대부분의 local minimum이 global minimum과 비슷한 loss 값**을 가지며, saddle point는 많지만 momentum 등으로 탈출할 수 있다는 것이 경험적으로 관찰됩니다. 이론과 실제의 간극이 여전히 큰 분야이지만, "실제로 잘 되니까 쓴다"가 현재 딥러닝의 현실입니다.</span>

---

## Normalization

### Batch Normalization (BatchNorm)

BatchNorm은 **같은 channel(feature) 방향**, 즉 batch 내 여러 샘플들을 묶어서 정규화한다. 각 feature column의 평균을 빼고 분산으로 나누는 방식이다. batch dimension을 활용하기 때문에 batch 크기가 충분히 커야 효과적이다.

![img13](/assets/lecture_img/neural_network/img13.png)

### Layer Normalization (LayerNorm)

LayerNorm은 **각 샘플 내부의 feature들**, 즉 한 샘플 안에서 정규화한다. 한 샘플의 모든 feature 값에 대해 평균과 분산을 계산해 normalize한다. batch 크기에 덜 의존적이기 때문에 NLP나 Transformer 계열에서 주로 사용된다.

![img14](/assets/lecture_img/neural_network/img14.png)

### 요약

| | BatchNorm | LayerNorm |
|---|---|---|
| 정규화 기준 | 같은 feature를 여러 샘플에 걸쳐 | 한 샘플 안의 여러 feature를 |
| 주요 사용처 | CNN | Transformer, NLP |
| Batch 의존성 | 높음 | 낮음 |

<span style="color:#60a5fa">Normalization이 학습을 돕는 이유는 여러 가지로 설명됩니다. 원래 BatchNorm 논문은 **internal covariate shift**(층 사이 입력 분포 변화)를 줄인다고 주장했지만, 이후 연구에서는 실제 효과가 **loss surface를 부드럽게 만들어 gradient의 변동성을 줄이는 것**에 가깝다고 밝혔습니다. 그 결과 더 큰 learning rate를 쓸 수 있고, 초기화에 덜 민감해집니다. Transformer에서 LayerNorm이 표준이 된 이유는 sequence 길이가 가변적이고 batch 크기에 의존하지 않아야 하기 때문입니다. 최근에는 **RMSNorm**처럼 평균을 빼지 않고 분산만으로 정규화하는 더 간단한 변형도 널리 쓰입니다.</span>
