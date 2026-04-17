---
title: "Normalization"
course: basic-ai
course_title: "Basic of AI"
order: 5
description: "Batch Normalization부터 Layer/Instance/Group Norm까지, 정규화 기법의 원리와 FC/Conv에서의 동작 차이를 정리합니다."
mathjax: true
---

## Batch Normalization

Layer의 Output을 Normalization하여 **zero mean / unit variance**를 갖게 한다.

→ reduce **internal covariate shift**에의한 optimization을 위해

학습 가능한 파라미터 $\gamma$, $\beta$를 도입한다.

$$
\hat{x} = \frac{x - \mu}{\sqrt{\sigma^2 + \epsilon}}
$$

$$
y = \gamma \hat{x} + \beta
$$

- $\gamma$: 크기를 조절하는 scale 파라미터
- $\beta$: 위치를 조절하는 shift 파라미터

정규화하면 표현력이 제한될 수 있으니, 다시 적절한 크기의 값으로 조절할 수 있게 $\gamma$, $\beta$를 둔다.

즉, 고정되는 것이 아니라 모델이 다시 scale과 shift를 할 수 있게 해주는 것이다.

<span style="color:#60a5fa">Batch Norm은 train 때는 batch-dependent한 $\mu$와 $\sigma^2$를 쓰고, test 때는 학습한 전체 데이터의 running average를 사용합니다. 학습 중에는 매 batch마다 $\mu$, $\sigma^2$를 구하고, running mean/variance를 EMA(exponential moving average)로 누적합니다. 이로 인해 **train과 test의 behavior가 다르다**는 특성이 있고, 이것이 BatchNorm의 가장 큰 단점 중 하나입니다.</span>

### BatchNorm의 효과

- Internal covariate shift 감소
- **Loss surface를 부드럽게** 만들어 gradient의 변동성을 줄임
- 더 큰 learning rate 사용 가능
- 초기화에 덜 민감

### BatchNorm의 위치

보통 **FC나 Conv Layer 다음에, Activation 전에** 넣는다.

$$
\text{Conv/FC} \rightarrow \text{BatchNorm} \rightarrow \text{Activation}
$$

### Conv에서의 BatchNorm

Conv에서는 **Channel 단위**로 normalize한다.

- Train의 기울기: 높고 낮은 게 있다
- network가 수 기울기를 학습해서 더 안정적이고 robust하게 학습

---

## Layer Normalization

FC에서는 LayerNorm, Conv에서는 InstanceNorm이 사용된다.

- **BatchNorm**: 여러 이미지에 걸쳐 하나의 채널(feature)별로 normalize
- **LayerNorm**: 한 이미지 안에서 모든 채널(feature)에 걸쳐 normalize
- **InstanceNorm**: 이미지 한 장 안에서 채널 하나의 픽셀에 대해 normalize
- **GroupNorm**: 이미지 한 장 안에서 채널 일부 그룹을 정규화

---

## Fully-Connected Case

한 샘플 안에 있는 feature(즉, 그 벡터)를 기준으로 평균과 분산을 구해서 정규화한다.

- **Batch Norm**에서는 각 feature 열마다 평균과 표준편차를 구해서 정규화 → 여러 샘플에 걸쳐
- **Layer Norm**에서는 각 샘플의 행(row) 평균과 표준편차를 구해서 정규화 → 한 샘플 내에서

> 💡 LayerNorm은 한 샘플 안에서만 계산하기 때문에 **batch에 의존하지 않는다**. 따라서 train과 test에서 Same behavior. 이것이 Transformer 계열에서 LayerNorm이 표준인 이유이다.

---

## Conv Case

| | 정규화 기준 | 축 |
|---|---|---|
| **Batch Norm** | 같은 채널을 여러 이미지에 걸쳐 | N, H, W |
| **Layer Norm** | 한 이미지 안의 모든 채널과 공간 | C, H, W |
| **Instance Norm** | 한 이미지, 한 채널의 공간 | H, W |
| **Group Norm** | 한 이미지, 채널 그룹의 공간 | C/G, H, W |

<span style="color:#60a5fa">각 Norm의 사용처를 정리하면: **BatchNorm**은 CNN + 큰 batch에서 가장 효과적이며 ResNet 등의 표준입니다. **LayerNorm**은 Transformer/NLP에서 표준이며 batch size에 무관합니다. **InstanceNorm**은 Style Transfer에서 주로 쓰이는데, 스타일 정보가 채널별 통계량에 담겨 있기 때문에 이를 정규화하면 스타일을 제거하는 효과가 있습니다. **GroupNorm**은 batch size가 작을 수밖에 없는 detection/segmentation에서 BatchNorm의 대안으로 사용됩니다. Group 수 G=1이면 LayerNorm, G=C이면 InstanceNorm과 동일합니다.</span>

---

## 핵심 비교

| | BatchNorm | LayerNorm | InstanceNorm | GroupNorm |
|---|---|---|---|---|
| 정규화 기준 | 같은 채널, 여러 샘플 | 한 샘플, 모든 채널 | 한 샘플, 한 채널 | 한 샘플, 채널 그룹 |
| Batch 의존 | 높음 | 없음 | 없음 | 없음 |
| Train/Test 차이 | 있음 | 없음 | 없음 | 없음 |
| 주요 사용처 | CNN (ResNet 등) | Transformer, NLP | Style Transfer | Detection, 작은 batch |
