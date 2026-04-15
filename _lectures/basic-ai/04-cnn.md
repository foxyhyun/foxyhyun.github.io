---
title: "Convolutional Neural Network"
course: basic-ai
course_title: "Basic of AI"
order: 4
description: "1D/2D Convolution부터 파라미터 공유, Shift Equivariance, Sparsity까지 CNN의 핵심 원리를 정리합니다."
mathjax: true
---

## 1D Convolution

![img1](/assets/lecture_img/cnn/img1.png)

이렇게 각 연산을 하다보면 값이 2로 커져도 결국에 필터 연산을 하게 되면 output은 0이 된다. 하지만 1에서 2로 넘어가는 지점, 즉 **값이 바뀌는 지점**에서는 output에도 변화가 생긴다.

이걸 바로 **Edge**라고 부르고, 각각 Edge Extractor, feature가 되는 것이다.

![img2](/assets/lecture_img/cnn/img2.png)

<span style="color:#60a5fa">1D convolution은 신호 처리의 기본 연산입니다. 필터(커널)가 상수 영역에서는 0을 출력하고, 값이 급격히 변하는 지점에서만 큰 반응을 보이기 때문에 **차이(difference)를 감지하는 연산**으로 이해할 수 있습니다. 이는 미분과 유사한 성질로, 이후 2D로 확장되면 이미지의 edge를 감지하는 역할을 하게 됩니다.</span>

---

## 2D Convolution

![img3](/assets/lecture_img/cnn/img3.png)

![img4](/assets/lecture_img/cnn/img4.png)

입력 이미지의 작은 부분을 필터와 곱하여 다 더하면 출력 픽셀이 된다. 사실 2D나 1D나 크게 다르지 않다.

1D에서도

$$
\text{output} = \text{Input} * \text{Filter}, \quad y = W * X
$$

였던 것이고, 그걸 확장한 것이 2D이다.

- $n, m$ : 현재 보고 있는 위치
- $i, j$ : 그 주변의 상대 위치
- $r$ : 커널 반지름, kernel size $= 2r + 1$

![img5](/assets/lecture_img/cnn/img5.png)

전체적인 그림은 다음과 같다.

![img6](/assets/lecture_img/cnn/img6.png)

![img7](/assets/lecture_img/cnn/img7.png)

- **입력 이미지**: $3 \times 64 \times 64$
- **필터**: $16 \times 3 \times 3 \times 3$ (16개의 필터, 3개의 채널)
- **출력**: 16개의 채널을 가진 $64 \times 64$ 이미지

> 💡 RGB 각 채널별로 $64 \times 64$로 총 3장이 있다. 필터가 $16 \times 3 \times 3 \times 3$이라면 16개의 $3 \times 3 \times 3$ 필터가 있는 것인데, 이는 RGB 각 채널별로 $3 \times 3$짜리 필터가 있는 것이다. **입력 채널 수 = 필터 깊이**와 맞아야 하고, **출력 채널 수 = 필터 개수**와 맞아야 한다. 쉽게 말해서 3개의 RGB 채널을 더 다양하게 16개의 채널로 확장하는 것이다.

![img8](/assets/lecture_img/cnn/img8.png)

**파라미터**:

- **Weights**: $C_o \times C_i \times K_h \times K_w = 16 \times 3 \times 3 \times 3$ ⇒ 필터와 동일
- **Bias**: $C_o = 16$ ⇒ 필터 개수와 동일

**FLOPs**: $\text{Params} \times H_o \times W_o$

**Params per FLOP**: $1 / (\text{spatial size}) = 1 / (64 \times 64)$

즉, **CNN은 파라미터를 공유하기 때문에 필요한 파라미터 수가 적다.**

<span style="color:#60a5fa">Fully-Connected Layer였다면 $64 \times 64 \times 3$ 입력에서 $64 \times 64 \times 16$ 출력으로 가는 데 약 8억 개의 파라미터가 필요합니다. 반면 CNN은 같은 연산을 **432개(16×3×3×3)의 weight + 16 bias**만으로 처리합니다. 이 엄청난 차이가 CNN을 이미지 처리의 표준으로 만든 핵심 이유입니다.</span>

---

## Padding과 출력 크기

**패딩을 왜 해야 할까?** 층을 여러 개 쌓으면 패딩 없이는 feature map 크기가 너무 빨리 줄어들기 때문이다.

$$
H_{out} = \left\lfloor \frac{H_{in} + 2 \cdot pad_h - K_h}{\text{stride}} + 1 \right\rfloor
$$

**예시**) Input: 224, padding: 3, kernel size: 7, stride: 2

$$
(224 + 2 \times 3 - 7) / 2 + 1 = 112
$$

---

## Shift Equivariance

![img9](/assets/lecture_img/cnn/img9.png)

이미지를 먼저 이동시키고 convolution 하든, 먼저 convolution 하고 결과를 이동시키든, **결과가 같다**.

→ 컨볼루션은 사진 위를 훑는 도장이나 탐지기 같아서, 물체가 움직이면 탐지 결과도 똑같이 같이 움직인다.

<span style="color:#60a5fa">이 성질을 **Translation Equivariance(이동 등변성)** 라고 합니다. "같은 필터가 이미지 모든 위치에서 동일하게 적용된다"는 convolution의 구조 자체가 이 성질을 보장합니다. 덕분에 CNN은 고양이가 이미지 왼쪽에 있든 오른쪽에 있든 같은 feature로 인식할 수 있습니다. 이는 이미지 데이터가 가진 자연스러운 대칭성을 모델 구조에 반영한 **inductive bias**의 대표적인 예입니다.</span>

---

## Sparsity와 Weight Sharing

![img10](/assets/lecture_img/cnn/img10.png)

Convolution은 주변에만 연결하고 같은 weight를 반복 사용한다. 즉, **파라미터 수가 가장 적다.**

![img11](/assets/lecture_img/cnn/img11.png)

보면 늘 새로운 weight를 쓰는 것이 아닌 $w_1, w_2, w_3$ 위치만 바꿔서 사용 중이다.

즉, **weight-sharing**이라는 말은 "서로 다른 출력 위치들이 같은 파라미터를 공유한다"는 뜻이다.

Convolution은 출력 하나가 입력 전체를 보는 게 아니라, 입력의 **일부분만 본다**. 따라서 다른 곳은 0일 수 있는 **Sparse Matrix**이다.

즉, 정리하면 **Fully-Connected Layer보다 Sparse하고 가중치를 공유하기 때문에 효율적인 네트워크**이다.

<span style="color:#60a5fa">CNN의 효율성은 두 가지 구조적 제약에서 나옵니다. 첫째, **Local Connectivity(Sparsity)**: 출력 뉴런 하나는 입력의 작은 local 영역(receptive field)만 봅니다. 이미지에서 가까운 픽셀끼리 강한 상관관계를 가진다는 사실을 반영한 설계입니다. 둘째, **Weight Sharing**: 같은 필터가 이미지 전체에 재사용됩니다. 이는 "edge detector가 이미지 왼쪽에서 유용하다면 오른쪽에서도 유용하다"는 가정과 같습니다. 이 두 가지 덕분에 CNN은 FC layer 대비 수백~수천 배 적은 파라미터로도 훨씬 강력한 성능을 냅니다.</span>
