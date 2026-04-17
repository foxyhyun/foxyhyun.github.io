---
title: "Visual Representations"
course: computer-vision
course_title: "Computer Vision"
order: 5
description: "Data Preprocessing, Weight Initialization, Data Augmentation, Regularization, Transfer Learning까지 시각 표현 학습의 핵심을 정리합니다."
mathjax: true
---

## Data Preprocessing

![img1](/assets/lecture_img/visual_representations/img1.png)

각 feature마다 평균을 빼서 중심을 0으로 옮긴다. 이후 각 feature를 std로 나눠서 스케일을 맞춘다.

음수도 있고 양수도 있으니까 weight 업데이트가 지그재그로 흔들릴 수 있다. 따라서 **Normalization**이 그런 걸 잡는 데 꽤 도움이 된다.

- **PCA**를 하면 feature들 사이가 decorrelated가 됨
- **Whitening**하여 각 축의 스케일을 맞춰 모든 축의 분산이 1이 되게 만듦. 분산을 1로 정규화하는 것

<span style="color:#60a5fa">Zero-centering을 하지 않으면 모든 입력이 양수가 되어, gradient가 항상 같은 부호로만 업데이트되는 문제가 생깁니다. 이것이 학습 시 지그재그 현상의 원인입니다. 실무에서는 ImageNet 전체의 채널별 평균과 표준편차(mean=[0.485, 0.456, 0.406], std=[0.229, 0.224, 0.225])를 빼고 나누는 것이 거의 표준입니다. PCA/Whitening은 이론적으로 좋지만 이미지에서는 거의 사용하지 않고, 주로 tabular 데이터에서 쓰입니다.</span>

---

## Weight Initialization

신경망의 가중치를 처음에 어떤 값으로 시작할 것인가?

- **0으로 두면?** 모든 뉴런이 똑같이 행동해서 서로 다른 걸 못 배움
- **작은 랜덤값?** 조금 낫지만, 깊은 네트워크에서는 신호가 점점 작아짐
- **큰 랜덤값?** activation이 saturate돼서 gradient가 거의 안 흐름

Gradient가 0이면 weight가 업데이트 되지 않아서 학습이 안 된다. 그렇다고 weight를 크게 초기화하면 tanh에 의해 -1, 1에 붙어버린다. 이걸 **saturate**라고 부른다.

### Xavier Initialization

$$
W = \frac{\text{randn}(D_{in}, D_{out})}{\sqrt{D_{in}}}
$$

랜덤하게 뽑되, 입력 차원 $D_{in}$이 클수록 weight 크기를 자동으로 줄인다.

- Tanh에서 잘 됨
- ReLU가 음수를 0으로 만들기 때문에 activation이 다시 작아짐 → **He Initialization** 사용해야 함

### He (Kaiming/MSRA) Initialization

ReLU를 쓸 때는 Xavier 대신 **Kaiming/MSRA Initialization**을 사용해서 activation의 크기가 layer가 깊어져도 유지된다. 분모에 2가 들어가서 scale이 좀 더 커진 것이 차이다.

<span style="color:#60a5fa">Xavier는 $\text{Var}(W) = 1/D_{in}$, He는 $\text{Var}(W) = 2/D_{in}$입니다. ReLU가 음수를 0으로 만들면 출력의 분산이 절반으로 줄기 때문에, 이를 보상하기 위해 2를 곱하는 것입니다. 이 간단한 차이가 깊은 ResNet 학습의 성패를 가릅니다. 현대에는 초기화보다 **BatchNorm/LayerNorm**이 activation 스케일을 자동으로 조절해주기 때문에 초기화의 중요성이 약간 줄었지만, 여전히 올바른 초기화는 학습 초반 안정성에 큰 영향을 줍니다.</span>

---

## Data Augmentation

- **Horizontal Flips**
- **Random Crops and Scales**
    - 학습할 때는 한 장의 이미지를 그대로 넣지 않고, 매번 랜덤하게 바꿔서 넣음
    - 여러 이미지 scale로 resize하고 전체 평균
- **Color Jitter**
    - Apply PCA to all RGB pixels

<span style="color:#60a5fa">Data Augmentation은 사실상 **무료로 데이터를 늘리는 것**과 같습니다. 핵심 원칙은 "augmentation 후에도 label이 변하지 않아야 한다"는 것입니다. 고양이 사진을 좌우 반전해도 여전히 고양이지만, 숫자 6을 상하 반전하면 9가 되므로 주의가 필요합니다. Color Jitter에서 PCA를 적용하는 이유는, RGB 채널 간 자연스러운 상관관계를 유지하면서 색상을 변형하기 위함입니다. 이는 AlexNet 논문에서 처음 제안된 기법입니다.</span>

---

## Regularization

Loss에 추가항을 넣어서 강도를 조절한다.

### L1 Regularization

$$
R(W) = \sum_k \sum_l |W_{k,l}|
$$

- Weight를 작게 만들지만, L2보다 많은 weight를 0쪽으로 밀어붙이기 때문에 **불필요한 weight를 아예 없애는 방향**

### L2 Regularization

$$
R(W) = \sum_k \sum_l W_{k,l}^2
$$

- 큰 Weight에 더 큰 벌점을 줘서 Weight를 전체적으로 작게 유지 (**Weight Decay**)

### Elastic Net

$$
R(W) = \sum_k \sum_l \beta W_{k,l}^2 + |W_{k,l}|
$$

- L1의 0으로 보내는 효과
- L2의 부드럽게 줄이는 효과
- 위 두 개를 더한 방식

### Training vs Testing 전략

**For Training**: 랜덤성을 넣어 불확실하고 흔들리는 상황을 넣어줌 → Overfit 방지

**For Testing**: 그 랜덤성을 평균냄

### 주요 Regularization 기법들

- **Dropout**: 뉴런의 출력을 0으로 꺼버리는 방식
- **DropConnect**: 뉴런 사이 Weight 자체를 랜덤하게 0으로 만듦 (노드 자체는 남겨두고 연결선을 끊음)
    - Training: 연결선 끊음
    - Testing: 모든 연결선 사용
- **Fractional Pooling**
    - Training: 랜덤화된 풀링 사이즈 → 매번 다른 크기
    - Testing: 여러 샘플링 결과 평균
- **Stochastic Depth**
    - Training: Skip some residual blocks in ResNet
    - Testing: Use Whole Network
- **Cutout**: Set random image regions to 0
- **Mixup**: 두 이미지를 픽셀 단위로 섞어 새로운 이미지를 만듦
- **CutMix**: Mixup은 이미지를 투명하게 섞고, CutMix는 일부 패치를 다른 이미지의 패치로 교체함

<span style="color:#60a5fa">이 기법들의 공통점은 **학습 시 의도적으로 정보를 훼손**하여 모델이 특정 feature에 과의존하지 않게 하는 것입니다. Dropout은 "어떤 뉴런이 꺼져도 잘 작동해야 한다"는 제약을 주고, Stochastic Depth는 "어떤 층이 빠져도 작동해야 한다"는 더 강한 제약을 줍니다. CutMix가 Cutout이나 Mixup보다 일반적으로 더 좋은 이유는, 가려진 영역에도 유의미한 정보(다른 이미지)가 들어가기 때문에 학습 신호가 낭비되지 않기 때문입니다.</span>

---

## Representation Learning

**How to Represent Images?**

- 예전에는 Edge, orientation 등을 직접 설계해왔음
- 딥러닝은 비슷한 형태의 일반 모듈을 여러 층 쌓아두고, 그 안의 파라미터를 학습
- 층이 깊어질수록 더 작은 feature에서 큰 feature로 감 → 이것이 **Hierarchical Representation**

**Deep Representations are Transferable!**

큰 데이터셋에서 먼저 네트워크를 학습시키면 그 과정에서 배운 representation이 다른 문제에도 잘 통한다.

<span style="color:#60a5fa">초기 층은 edge, corner, color blob 같은 **low-level feature**를, 중간 층은 texture, pattern 같은 **mid-level feature**를, 깊은 층은 얼굴, 바퀴 같은 **high-level feature**를 학습합니다. 흥미로운 점은 초기 층의 feature는 task에 관계없이 거의 동일하다는 것입니다. 어떤 이미지 문제든 edge detector는 필요하니까요. 이 관찰이 Transfer Learning의 이론적 토대가 됩니다.</span>

---

## Transfer Learning

### Pretraining

큰 데이터에서 모델을 오래 학습시켜서 일반적인 표현을 배우게 한다.

- Large scale
- Long time for training

### Fine-tuning

이미 배운 weight를 가져와서 내 데이터셋, 내 task에 맞게 조금 조정하는 과정이다. 이미 좋은 표현을 배운 상태라서 처음부터 크게 흔들 필요가 없다.

- 적은 time
- 작은 Learning Rate

→ 여기서 weight를 처음부터 크게 업데이트하게 해주면, 그냥 처음부터 다시 학습하는 것과 같다.

### Partial Transfer

![img2](/assets/lecture_img/visual_representations/img2.png)

Pre-training과 target 데이터는 다를 수 있다. 그러면 앞쪽 층이 배운 edge나 texture 같은 건 여전히 유용할 수 있지만, 뒤쪽 층의 아주 고수준 feature는 원래 pre-training task에 너무 맞춰져 있을 수 있다.

따라서 **앞쪽 층 일부는 가져오고 뒤쪽 layer는 새로 초기화**해서 target task에 맞게 다시 학습한다.

### Frozen Weights

![img3](/assets/lecture_img/visual_representations/img3.png)

Pre-trained layer의 weight를 **업데이트하지 않고 그대로 두는 것**이다.

- 데이터가 너무 적을 때 적은 데이터로 모든 가중치를 다 바꾸면 오히려 overfit. 그래서 일부 층 고정해서 안정적으로 사용
- 계산량 줄이기
- Feature extractor처럼 사용

즉, **이미 잘 배운 부분은 건드리지 말고, 필요한 부분만 학습하자**는 전략이다.

### Network Surgery

![img4](/assets/lecture_img/visual_representations/img4.png)

원래 다른 목적으로 pre-train한 backbone을 다른 task에도 가져다 쓸 수 있다. 따라서 **앞쪽 backbone은 유지하고, 뒤에 붙는 head만 바꿔서** 다른 task로 재활용한다.

<span style="color:#60a5fa">Transfer Learning의 전략은 **데이터 양과 도메인 유사성**에 따라 달라집니다. 데이터가 적고 도메인이 비슷하면 → Frozen weights + linear head만 학습. 데이터가 적고 도메인이 다르면 → 앞쪽 층만 freeze하고 뒤쪽은 새로 학습. 데이터가 많으면 → 전체 fine-tuning이 가능하지만 작은 learning rate로 시작. 현대에는 Foundation Model(CLIP, DINOv2 등)이 범용 feature extractor 역할을 하면서, transfer learning의 중요성이 더욱 커졌습니다.</span>

---

## How Can We Train Good Representations?

### 1. Model

- 더 깊고
- 더 넓고
- 더 큰 capacity
- Inductive bias: convolution, recurrency, attention

### 2. Data

- 데이터 양
- Noisy label
- 품질 낮고 편향된 샘플

### 3. Learning Objective

- Supervised
- Unsupervised
- Self-supervised

<span style="color:#60a5fa">최근 연구의 가장 큰 트렌드는 **Self-Supervised Learning(SSL)** 입니다. Label 없이 데이터 자체에서 학습 신호를 만들어내는 방식으로, MAE(Masked Autoencoder), DINO, SimCLR 등이 대표적입니다. SSL로 pre-train한 모델이 supervised pre-training보다 더 좋은 transfer 성능을 보이는 경우도 많아졌습니다. 이는 "좋은 representation이란 무엇인가"에 대한 근본적인 질문으로 이어지며, 현재 vision 연구의 핵심 주제 중 하나입니다.</span>
