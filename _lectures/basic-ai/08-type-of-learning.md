---
title: "Type of Learning"
course: basic-ai
course_title: "Basic of AI"
order: 8
description: "Supervised, Unsupervised, Self-Supervised Learning의 차이와 K-means, Autoencoder, PCA, Contrastive Learning, DINO, MAE까지 정리합니다."
mathjax: true
---

## 학습의 종류

학습 방식은 **label의 유무**에 따라 크게 세 가지로 나뉜다.

- **Supervised Learning**: 입력과 정답 label이 함께 주어짐. Classification, Regression 등
- **Unsupervised Learning**: label 없이 데이터의 구조나 패턴을 찾음. Clustering, Dimensionality Reduction 등
- **Self-Supervised Learning**: label 없이 데이터 자체에서 학습 신호를 만들어냄. Contrastive Learning, Masked Prediction 등

<span style="color:#60a5fa">Supervised Learning은 성능이 좋지만 대규모 label 데이터를 모으는 비용이 매우 큽니다. ImageNet은 수백만 장에 사람이 직접 label을 달았는데, 이런 방식은 확장성에 한계가 있습니다. 이 때문에 label 없이도 좋은 representation을 학습하는 Unsupervised/Self-Supervised 방법이 점점 중요해지고 있습니다.</span>

---

## Unsupervised Learning

### K-means Clustering

데이터를 **k개의 center**로 대표하는 방법이다.

![img1](/assets/lecture_img/typeoflearning/img1.png)

1. **Assignment**: 각 샘플을 가장 가까운 center에 할당
2. **Update**: 각 center를 할당된 샘플들의 평균으로 갱신

이 두 단계를 수렴할 때까지 반복한다.

### K-means와 Autoencoding의 관계

![img2](/assets/lecture_img/typeoflearning/img2.png)

K-means의 loss function은 다음과 같다.

$$
\min_{D, E} \sum_x \| x - D(E(x)) \|^2
$$

- **Encoder $E$**: $x$를 code(one-hot)로 매핑
- **Decoder $D$**: code를 center로 매핑

즉, **K-means는 Autoencoding과 같은 구조**이다. 입력을 압축된 표현으로 바꾸고, 다시 복원하는 과정에서 representation을 학습하는 것이다.

<span style="color:#60a5fa">K-means가 Autoencoder와 같다는 관점은 매우 중요합니다. K-means의 encoder는 "가장 가까운 center의 인덱스"를 출력하는 **discrete(one-hot) encoder**이고, decoder는 "그 인덱스에 해당하는 center 좌표"를 출력합니다. Autoencoder는 이를 **continuous하게 일반화**한 것으로, encoder와 decoder 모두 신경망으로 구성해 더 풍부한 표현을 학습합니다.</span>

---

## Dimensionality Reduction과 PCA

### Redundancy

![img3](/assets/lecture_img/typeoflearning/img3.png)

Feature들 사이에 **redundancy(중복성)** 가 높으면, 하나의 feature로 다른 feature를 거의 예측할 수 있다. 이 경우 $(r_1, r_2)$는 좋은 representation이 아니다.

![img4](/assets/lecture_img/typeoflearning/img4.png)

- **PC1**: 정보를 가장 많이 유지하는 방향 (retains most information)
- **PC2**: 거의 정보가 없는 방향 (could be dropped)

Redundancy가 높을수록 하나의 축(PC1)만으로도 데이터를 잘 표현할 수 있으므로, 차원을 줄여도 정보 손실이 적다.

<span style="color:#60a5fa">PCA(Principal Component Analysis)는 데이터의 분산이 가장 큰 방향을 찾아 그 축으로 투영하는 방법입니다. 수학적으로는 공분산 행렬의 고유벡터(eigenvector)를 구하는 것과 같습니다. 첫 번째 주성분(PC1)이 전체 분산의 대부분을 설명하면, 나머지 성분은 버려도 정보 손실이 거의 없습니다. 이는 Autoencoder의 linear 버전으로 볼 수 있으며, linear autoencoder의 최적해가 PCA와 동일하다는 것이 증명되어 있습니다.</span>

---

## Self-Supervised Learning

Label 없이 데이터 자체에서 학습 신호를 만들어내는 방식이다.

### Contrastive Learning Loss

![img5](/assets/lecture_img/typeoflearning/img5.png)

$$
L = -\mathbb{E}_X \left[ \log \frac{\exp(s(f(x), f(x^+)))}{\exp(s(f(x), f(x^+))) + \sum_{j=1}^{N-1} \exp(s(f(x), f(x_j^-)))} \right]
$$

- **분자**: positive pair의 score — 같은 이미지의 다른 augmentation끼리 유사도를 높임
- **분모**: positive pair + N-1개의 negative pair — negative와는 유사도를 낮춤

<span style="color:#60a5fa">Contrastive Loss는 본질적으로 **softmax cross-entropy**와 같은 구조입니다. "positive pair를 N개의 후보 중에서 맞추는 분류 문제"로 볼 수 있습니다. negative sample이 많을수록(N이 클수록) 학습 신호가 풍부해지지만, 그만큼 메모리와 계산량이 커집니다. 이 trade-off를 해결하기 위해 MoCo는 momentum queue를, SimCLR은 큰 batch를 사용합니다.</span>

---

### MoCo (Momentum Contrastive Learning)

![img6](/assets/lecture_img/typeoflearning/img6.png)

- **Positive pair의 유사도를 최대화**, negative pair의 유사도를 최소화
- **Query encoder**: BackProp으로 업데이트
- **Key encoder**: gradient 없이 **moving average**로 업데이트

$$
\theta_k \leftarrow m \cdot \theta_k + (1 - m) \cdot \theta_q
$$

Key encoder를 천천히 업데이트함으로써, 큰 queue에 저장된 과거 key들과의 일관성을 유지한다.

<span style="color:#60a5fa">MoCo의 핵심 아이디어는 **dictionary as a queue**입니다. Negative sample을 많이 확보하려면 큰 batch가 필요한데, GPU 메모리 한계가 있습니다. MoCo는 과거 mini-batch의 key를 queue에 저장해두고 재사용함으로써, 작은 batch로도 수만 개의 negative sample을 확보합니다. Momentum update($m$ ≈ 0.999)는 key encoder가 급격히 변하지 않게 해서 queue 안의 key들이 일관된 encoder로 만들어진 것처럼 보이게 합니다.</span>

---

### SimCLR (Simple Contrastive Learning)

![img7](/assets/lecture_img/typeoflearning/img7.png)

- **Positive pair의 유사도를 최대화**, negative pair의 유사도를 최소화
- 두 encoder가 **같은 weight를 공유**
- 두 encoder 모두 **BackProp으로 업데이트**

MoCo와 달리 momentum이나 queue 없이, 같은 batch 안의 다른 샘플들을 negative로 사용한다.

<span style="color:#60a5fa">SimCLR은 MoCo보다 구조가 단순하지만, **큰 batch size**(4096~8192)가 필요합니다. Batch가 커야 충분한 negative sample이 확보되기 때문입니다. 또한 SimCLR에서 중요한 발견은 **projection head**(encoder 뒤에 붙는 작은 MLP)가 성능에 큰 영향을 준다는 것입니다. Contrastive loss는 projection head의 출력에 적용하고, 실제 downstream task에는 projection head 이전의 representation을 사용합니다.</span>

---

### DINO

![img8](/assets/lecture_img/typeoflearning/img8.png)

- **Student**: 입력 이미지의 **Local View**(작은 crop)를 받음
- **Teacher**: 입력 이미지의 **Global View**(큰 crop)를 받음
- Student와 Teacher는 **같은 network architecture**를 사용

Student는 Teacher의 출력을 따라하도록 학습된다. Teacher는 Student의 EMA(exponential moving average)로 업데이트된다.

<span style="color:#60a5fa">DINO의 핵심 통찰은 **"local view만 보고도 global view의 정보를 예측할 수 있어야 한다"** 는 것입니다. 이미지의 작은 부분만 봐도 전체 이미지가 무엇인지 알 수 있으려면, 모델이 의미적(semantic) 표현을 배워야 합니다. DINO로 학습한 ViT의 attention map을 시각화하면, 별도의 supervision 없이도 **object segmentation에 가까운 결과**가 나온다는 것이 큰 화제가 되었습니다. Contrastive Learning과 달리 explicit negative sample이 필요 없다는 것도 장점입니다.</span>

---

### MAE (Masked Autoencoder)

![img9](/assets/lecture_img/typeoflearning/img9.png)

이미지를 patch로 나눈 뒤 대부분(75%)을 **masking**하고, 보이는 patch만 encoder에 넣는다. 이후 decoder가 **masked patch의 원래 픽셀을 복원**하도록 학습한다.

$$
L = \| x_{\text{masked}} - \hat{x}_{\text{masked}} \|^2
$$

Masked patch에 대해서만 **L2 loss in pixels**를 계산한다.

<span style="color:#60a5fa">MAE는 NLP의 BERT에서 영감을 받았지만, 이미지에서는 훨씬 높은 masking ratio(75%)가 필요합니다. 이는 이미지의 **정보 밀도가 텍스트보다 낮기** 때문입니다. 텍스트에서 단어 하나를 가리면 의미가 크게 달라지지만, 이미지에서 패치 하나가 빠져도 주변 패치로 쉽게 유추할 수 있습니다. 높은 masking ratio를 써야 모델이 단순한 interpolation을 넘어 **의미적 이해**를 하게 됩니다. 또한 encoder가 보이는 패치(25%)만 처리하므로 학습 속도가 매우 빠르다는 실용적 장점도 있습니다.</span>
