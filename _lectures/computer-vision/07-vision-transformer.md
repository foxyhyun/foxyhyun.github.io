---
title: "Vision Transformer"
course: computer-vision
course_title: "Computer Vision"
order: 7
description: "ViT의 구조와 Patch Embedding, Position Encoding, Swin Transformer, MLP-Mixer, Diff Transformer까지 정리합니다."
mathjax: true
---

## CNN + Self-Attention

![img17](/assets/lecture_img/transformer/img17.png)

CNN은 local pattern을 잘 잡아서 edge, texture, shape 같은 근처 정보를 잘 본다. 하지만 receptive field가 커지려면 layer를 많이 쌓아야 한다. 반면 self-attention은 멀리 있는 위치 관계를 잘 보기 때문에 **둘을 섞자**는 아이디어가 나왔다.

이미지의 해상도가 큰 초반과 후반에 넣게 되면 이미지 사이즈가 너무 커서 Attention의 계산량이 매우 크다. 따라서 **조금 낮은 Middle Point에 적용**하는 것이다.

Local attention은 convolution처럼 주변 local patch만 보되, 고정 커널 대신 현재 위치에 따라 달라지는 attention weights로 주변 정보를 합치는 방식이다.

- Convolution은 고정된 filter를 가지고 보는데, Attention은 고정된 filter 대신 **입력에 따라 달라지는 weight**를 사용한다
- Attention으로 convolution을 완전히 대체해보려 했지만, 생각만큼 깔끔하게 더 좋지는 않았다
- Transformer를 사용하려고 했지만, 픽셀 하나하나 하기에는 연산량이 너무 많았다

<span style="color:#60a5fa">이 시도들이 ViT 이전의 "CNN + Attention" 하이브리드 시기입니다. CBAM, Non-local Networks, Squeeze-and-Excitation 같은 방법들이 CNN 중간에 attention을 삽입했습니다. 하지만 CNN backbone 없이 attention만으로 이미지를 처리하려면 **패치 단위**로 토큰화하는 아이디어가 필요했고, 그것이 ViT로 이어졌습니다.</span>

---

## ViT (Vision Transformer)

![img18](/assets/lecture_img/transformer/img18.png)

![img19](/assets/lecture_img/transformer/img19.png)

ViT는 이미지를 작은 패치들로 쪼개서 토큰처럼 본다. 각 패치를 펼쳐서 **Linear Projection**해서 벡터로 바꾸는데 이걸 **Patch Embedding**이라고 한다.

![img20](/assets/lecture_img/transformer/img20.png)

각 patch token에 위치 정보 **position embedding**을 더한다.

![img21](/assets/lecture_img/transformer/img21.png)

이렇게 만든 patch token을 Transformer Encoder에 넣는다. NLP Transformer와 동일한데 여기에 **Classification token**을 넣는다.

![img22](/assets/lecture_img/transformer/img22.png)

Transformer 출력 중에서 classification token에 해당하는 출력 벡터를 꺼내서 마지막 linear layer에 넣으면 클래스 점수(logits)가 나온다.

하지만, Self-Attention 자체는 입력의 순서를 고려하지 않기 때문에 Patch Embedding에서 **순서와 위치 정보를 더해줘야** Transformer에서 공간적 배치를 이해할 수 있다.

<span style="color:#60a5fa">ViT의 핵심 통찰은 **"이미지도 결국 패치 시퀀스로 보면 NLP와 같은 구조로 처리할 수 있다"** 는 것입니다. 16×16 패치 하나가 NLP에서의 단어 토큰 하나에 대응됩니다. Classification token([CLS])은 BERT에서 가져온 아이디어로, 모든 패치의 정보를 종합하는 역할을 합니다. 이 토큰의 최종 표현이 이미지 전체를 대표하게 됩니다.</span>

---

## Position Encoding

![img23](/assets/lecture_img/transformer/img23.png)

입력 $x$ 토큰은 내용만 있지 위치, 순서에 대한 정보는 없다. 그래서 **Position Encoding**을 사용해서 순서와 위치 정보를 Self-Attention에 넣어주는 것이다.

두 가지 방법이 있다.

### 1. Learned Lookup Table

- 위치마다 파라미터(embedding)를 학습
- 학습한 길이보다 더 긴 문장에는 일반화가 어려움
- 위치 수만큼 파라미터 필요

위치마다 벡터를 그냥 학습한다. 예를 들어 1번 위치용 벡터, 2번 위치용 벡터를 파라미터로 둔다. 직관적이지만 보통 학습한 길이보다 더 긴 시퀀스에는 약할 수 있다.

### 2. Fixed Function

- 위치를 사인/코사인 함수로 바꿔서 벡터로 만듦
- "Attention is All You Need"에서 사용
- 값이 bounded $[-1, 1]$
- Deterministic

Fixed Function이 Lookup Table에 비해 가지는 장점:

- 각 위치마다 서로 다른 값이 나와야 함
- 두 위치 사이의 관계가 문장 길이가 달라도 일관적
- 값이 bounded되어야 함
- 더 긴 문장에도 자연스럽게 일반화
- 랜덤하면 안 됨

<span style="color:#60a5fa">실제로 ViT 원 논문에서는 **learned position embedding**을 사용했고, fixed sinusoidal과 성능 차이가 거의 없었습니다. 흥미로운 점은 학습된 position embedding을 시각화하면 **가까운 패치끼리 비슷한 embedding**을 가지는 2D 구조가 자연스럽게 나타난다는 것입니다. 모델이 명시적으로 2D 정보를 주지 않아도 스스로 공간 구조를 학습하는 셈입니다.</span>

---

## ViT 학습 기법

아래 방법들을 사용해야 성능이 올라간다.

### Regularization

- Weight Decay
- Stochastic Depth
- Dropout

### Data Augmentation

- MixUp
- RandAugment

### Distillation

![img24](/assets/lecture_img/transformer/img24.png)

**Step 1: Teacher 모델 먼저 학습**

- 원래 데이터와 GT label로 먼저 학습

**Step 2: Student가 Teacher 따라 배움**

- 정답 label만 맞추는 게 아니라 teacher의 출력 분포 자체를 따라 하도록 학습

Student는 GT의 **Cross-Entropy Loss**, Teacher prediction의 **KL Divergence Loss**를 사용한다.

<span style="color:#60a5fa">ViT가 CNN보다 **더 많은 regularization과 augmentation**을 필요로 하는 이유는 inductive bias의 부재 때문입니다. CNN은 locality와 translation equivariance가 구조에 내장되어 있어 적은 데이터로도 합리적인 feature를 학습하지만, ViT는 이런 사전 가정 없이 모든 것을 데이터에서 배워야 합니다. DeiT 논문에서 Distillation이 특히 효과적이었던 이유는, **CNN teacher가 가진 local feature 추출 능력**을 ViT student에게 전달할 수 있기 때문입니다.</span>

---

## CNN vs ViT 구조 비교

CNN은 계층적으로 해상도를 줄이고 채널을 늘리는데, 기본 ViT는 거의 같은 해상도와 같은 차원 수를 유지한다.

- **CNN**: Hierarchical Architecture
- **ViT**: Isotropic Architecture

---

## Swin Transformer

기본 ViT처럼 모든 block이 같은 해상도를 유지하지 않고, stage를 거치면서 **해상도는 줄이고 channel 수는 늘리는 hierarchical 구조**를 가진다.

![img25](/assets/lecture_img/transformer/img25.png)

- **Patch Partition**: 입력을 $4 \times 4$ Patch로 나눔
- **Linear Embedding**: $4 \times 4$ Patch 하나를 벡터로 보고 그걸 $C$ 차원으로 projection. 즉, $C \times \frac{H}{4} \times \frac{W}{4}$
- **Swin Transformer Block**: 해상도 유지, local window attention을 통해 각 patch token들이 서로 정보를 주고 받음
- **Patch Merging**: CNN의 Downsampling 역할. $2 \times 2$ 이웃 patch 4개를 묶음. 전체적으로 조각이 줄어듦(해상도 감소). 4개의 patch를 합치니 $C \Rightarrow 4C$. 그렇지만 너무 클 수 있으니 Linear Projection으로 $2C$로 줄임

![img26](/assets/lecture_img/transformer/img26.png)

**예시)** Input image가 $224 \times 224$라고 한다면, patch grid는 $56 \times 56$개의 작은 patch가 생긴다.

1. $C \times 56 \times 56$
2. Patch Merging: $4C \times 28 \times 28 \Rightarrow 2C \times 28 \times 28$
3. Patch Merging: $8C \times 14 \times 14 \Rightarrow 4C \times 14 \times 14$
4. Patch Merging: $16C \times 7 \times 7 \Rightarrow 8C \times 7 \times 7$

즉, 처음에는 이미지를 아주 작게 여러 패치로 보고, 뒤로 갈수록 여러 패치를 묶어서 더 크게 보는 것이다. 또 채널도 늘려 더 깊고 다양하게 본다.

### Window Attention

Swin은 **전체를 다 보지 말고, 가까운 칸들끼리만 attention 하자**는 아이디어다.

원래는 $(HW) \times (HW)$인데 $M \times M$ 윈도우로 나누면 Window 하나는 $M^2$개의 토큰을 가지고 있다. 따라서 Window 하나의 연산량은 $(M^2) \times (M^2) = M^4$이다.

전체 이미지에는 $\frac{H}{M} \times \frac{W}{M}$개의 window가 있고 각 window에는 $M^4$의 연산량이 있으므로:

$$
M^4 \times \frac{H}{M} \times \frac{W}{M} = M^2 \cdot HW
$$

이미지 $H, W$에 대해 **선형적으로 증가**한 것이다. 반면 full attention은 $(HW)^2$으로 훨씬 더 연산량이 크다.

한계로는 같은 window 안에서만 attention하지, **다른 window에 있는 patch끼리는 직접 상호작용을 못한다.**

### Shifted Window

![img27](/assets/lecture_img/transformer/img27.png)

처음 block에서는 같이 있지 않던 patch들을 **window를 옮겨** 경계를 넘는 정보 전달을 가능케 한다.

### Relative Position Bias

보통 ViT는 Positional Embedding을 하지만 Swin은 Patch들 사이의 **상대적 위치(Relative Position)** 를 반영한다. 즉 이미지 전체 어디냐보다는 두 patch가 서로 얼마나 떨어져 있는지를 준다.

일반적인 Attention:

$$
A = \text{Softmax}\left(\frac{QK^T}{\sqrt{D}}\right)V
$$

Swin:

$$
A = \text{Softmax}\left(\frac{QK^T}{\sqrt{D}} + B\right)V
$$

여기서 $B$는 **relative positional bias**이다. 두 patch의 상대적인 위치에 따라 attention score를 조금 올리거나 내리는 역할을 한다.

<span style="color:#60a5fa">Swin Transformer는 ViT의 두 가지 핵심 약점을 해결합니다. 첫째, **계산 복잡도**: 전체 이미지에 대한 full attention 대신 window 단위로 제한해 $O((HW)^2)$를 $O(M^2 \cdot HW)$로 줄였습니다. 둘째, **hierarchical feature**: CNN처럼 다양한 해상도의 feature map을 만들어 object detection, segmentation 같은 dense prediction task에 바로 적용할 수 있습니다. Shifted window는 간단하면서도 효과적인 트릭으로, window 경계 문제를 거의 완벽히 해결합니다.</span>

---

## ViT의 역할과 MLP-Mixer

ViT에서 각 구성 요소의 역할을 정리하면 다음과 같다.

**1. Token끼리 섞기**

- 어떤 patch가 다른 patch 정보를 보게 하기 위해 Self-Attention을 한다.

**2. 각 Token 내부 변환**

- 정보가 섞인 뒤에, 각 token 벡터 자체를 한 번 더 가공한다.
- 각 token마다 똑같은 MLP를 사용하여 각 patch feature를 비선형 변환한다.

### Patch들 사이를 섞는 데 꼭 Self-Attention으로 해야 할까?

MLP를 사용해서 섞기 → **MLP-Mixer**

MLP-Mixer는 patch 내부를 섞는 MLP 하나, patch들끼리 섞는 MLP 하나를 써서, 결과적으로 CNN 비슷한 역할을 한다.

<span style="color:#60a5fa">MLP-Mixer는 "Attention이 정말 필요한가?"라는 근본적인 질문에서 나왔습니다. 결론적으로 MLP만으로도 꽤 좋은 성능이 나왔지만, 같은 크기에서는 ViT보다 약간 뒤처졌습니다. 이 실험이 의미하는 바는, **token mixing(패치 간 정보 교환) 자체가 중요한 것이지 그 방법이 반드시 attention이어야 하는 것은 아니다**라는 점입니다. 이후 ConvNeXt처럼 "Transformer의 설계 원칙을 CNN에 적용"하는 연구로도 이어졌습니다.</span>

---

## Transformer의 단점

Attention에서 softmax할 때 불필요한 토큰에도 어느 정도 weight가 퍼질 수 있다. → **Attention noise**

**Diff Transformer**라는 것이 있는데 이건 attention 두 개 만든 뒤, **보고 싶은 attention − 빼고 싶은 attention** 하자는 아이디어이다.

![img28](/assets/lecture_img/transformer/img28.png)

<span style="color:#60a5fa">Softmax의 특성상 모든 위치에 최소한의 weight가 분배됩니다. 토큰 수가 많아질수록 이 noise의 총량도 커지는데, 이것이 긴 시퀀스에서 attention 품질이 떨어지는 원인 중 하나입니다. Diff Transformer는 두 attention map의 차이를 취함으로써 **공통적으로 높은 noise를 상쇄**하고, 진짜 중요한 관계만 남기는 효과를 냅니다. 이는 신호 처리에서의 differential amplifier와 유사한 원리입니다.</span>
