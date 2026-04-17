---
title: "Attention & Transformer"
course: computer-vision
course_title: "Computer Vision"
order: 3
description: "CNN과 Self-Attention의 차이부터 Attention 메커니즘, Transformer Encoder 구조까지 정리합니다."
mathjax: true
---

## CNN vs Self-Attention

Convolution은 주변만 보며 각 layer가 쌓이면서 receptive field가 높아지면서 점점 더 넓게 보는 구조이다.

반면 Self-Attention은 멀리 있는 것들도 attention score를 계산해서 모든 위치에서 구조적 연결이 가능하다.

> 💡 CNN은 locality라는 inductive bias를 가지기 때문에 처음부터 주변 정보 위주로 본다. 반면 Transformer의 self-attention은 모든 위치를 한 번에 볼 수 있어서 전역 관계를 잘 잡을 수 있다. 하지만 그만큼 고려해야 할 관계가 많아서 계산량이 커지고, 구조적 제약이 적어서 더 많은 데이터가 필요할 수 있다.

→ CNN은 Local Pattern이 중요하다는 가정을 한다. 반면에 Transformer는 그런 가정이 없어서 더 유연한 대신에 더 많은 데이터를 봐야 한다.

---

## Attention

![img1](/assets/lecture_img/transformer/img1.png)

우리가 Citron을 검색하면 pomme, banane에서 실패하고 citron에서 성공하여 lemon을 출력한다.

![img2](/assets/lecture_img/transformer/img2.png)

하지만 신경망에서는 이렇게 하지 않고 **Embedding(숫자 벡터)** 으로 표현한다.

**즉, Attention은 문자 일치 검색이 아닌 의미적으로 비슷한 숫자 벡터(representation)을 검색하는 것이다.**

![img3](/assets/lecture_img/transformer/img3.png)

### Step 1. Query-Key Similarity

$$
s_i = q \cdot k_i
$$

현재 query가 각 key와 얼마나 비슷한가를 계산한다.

### Step 2. Attention Weight

![img4](/assets/lecture_img/transformer/img4.png)

$$
w_i = \frac{e^{s_i}}{\sum_j e^{s_j}}
$$

유사도 점수 $s_i$를 바로 쓰는 게 아니라 **얼마나 참고할지의 비율**로 바꾼다.

### Step 3. Weighted Average

![img5](/assets/lecture_img/transformer/img5.png)

$$
z = \sum_i w_i v_i
$$

특정 value를 바로 고르는 것이 아닌 attention weight에 value를 곱해서 output을 만든다.

이것들을 모든 위치에 동시에 multiple query를 하면 **self-attention**이 되는 것이다.

<span style="color:#60a5fa">Attention의 핵심은 **"hard selection(하나 골라)" 대신 "soft selection(비율로 섞어)"** 을 한다는 것입니다. 사전에서 단어를 정확히 찾는 게 아니라, 모든 항목과의 유사도를 계산해 가중 평균을 내는 방식입니다. 이 soft한 연산 덕분에 gradient가 모든 위치로 흐를 수 있어 end-to-end 학습이 가능합니다.</span>

---

## Einsum

![img6](/assets/lecture_img/transformer/img6.png)

정리하면 위와 같다.

"좋아, 행렬곱은 알겠는데, 더 복잡한 텐서 연산은 어떻게 한 줄로 표현하지?"

**Einsum**은 행렬곱, 합산, attention 계산을 인덱스 관점으로 더 압축해서 쓰는 방법이다. 어떤 축을 맞춰서 곱하고, 어떤 축을 합하고, 어떤 축을 남길지를 인덱스 문자만으로 쓴다.

![img7](/assets/lecture_img/transformer/img7.png)

### 규칙 1

**같은 문자가 여러 입력에 동시에 나오면 그 축끼리 맞춰서 곱한다.**

### 규칙 2

**입력에는 있었는데 출력에 안 쓰인 문자는 합(sum)해서 없앤다.**

기본적인 틀은 다음과 같다.

```python
einsum('입력축들 -> 출력축들', ...)
```

```python
einsum('ij->j', X)   # shape: (10, 5) -> (5,)
einsum('ij->i', X)   # shape: (10, 5) -> (10,)
einsum('ij->', X)    # shape: (10, 5) -> scalar
```

```python
einsum('i,i->', A, B)   # (10,) x (10,) -> scalar
einsum('i,j->ij', A, B)   # (10,) x (5,) -> (10, 5)
einsum('i,i->i', A, B)   # (10,) x (10,) -> (10,)
```

Matrix-vector product:

```python
einsum('ik,k->i', A, B)   # (10, 4) x (4,) -> (10,)
einsum('ik,kj->ij', A, B)   # (10, 4) x (4, 5) -> (10, 5)
einsum('nik,nkj->nij', A, B)   # (n,10,4) x (n,4,5) -> (n,10,5)
```

Transpose / Permute:

```python
einsum('ij->ji', X)   # (10,5) -> (5,10)
einsum('nchw->nhwc', X)   # (n,3,256,256) -> (n,256,256,3)
```

### Attention을 Einsum으로 표현

![img8](/assets/lecture_img/transformer/img8.png)

**1. Query-Key Similarity, Attention Weight, Weighted Average**

```python
S = einsum('qc,kc->qk', Q, K)
W = softmax(S, dim=-1)
Z = einsum('qk,kd->qd', W, V)
```

**2. Expand to Multi-head**

```python
S = einsum('hqc,hkc->hqk', Q, K)
W = softmax(S, dim=-1)
Z = einsum('hqk,hkd->hqd', W, V)
```

**3. Batch**

```python
S = einsum('nhqc,nhkc->nhqk', Q, K)
W = softmax(S, dim=-1)
Z = einsum('nhqk,nhkd->nhqd', W, V)
```

### Attention 정리

- Query 개수와 Key 개수는 같지 않아도 됨
- Output 개수 = Query 개수
- Key D와 Value D는 달라도 됨
- Query D = Key D (연산 때문에)
- Value D = Output D
- **Attention is parameter-free**
- 모든 key와 모든 value는 연결되어 있음

<span style="color:#60a5fa">Einsum이 유용한 이유는 **차원이 복잡해질수록 명확해지기 때문**입니다. Multi-head + Batch까지 들어가면 `matmul`이나 `bmm`으로는 어떤 축을 곱하고 어떤 축을 남기는지 헷갈리기 쉽습니다. Einsum은 인덱스 문자 자체가 문서 역할을 하므로, 코드를 읽을 때 "n은 batch, h는 head, q는 query 위치, c는 channel"이라는 것이 바로 드러납니다.</span>

---

## Self-Attention

일반 Attention은 입력 Q, K, V가 각자 다른 위치에서 들어올 수 있었다. 그렇지만 Self-Attention은 입력 시퀀스가 들어오고 **각 위치마다 Q, K, V를 만든다**.

![img9](/assets/lecture_img/transformer/img9.png)

즉, Input → Q, K, V → Attention Block

- **Queries**: 지금 무엇을 찾고 싶은가
- **Keys**: 어떤 정보가 relevant한가
- **Values**: 실제로 가져올 내용은 무엇인가

![img10](/assets/lecture_img/transformer/img10.png)

시퀀스 길이가 7이면,

- 1번 위치의 query도 1~7번 key 전부를 보고
- 2번 위치의 query도 1~7번 key 전부를 보고
- ...
- 7번 위치의 query도 전부를 본다

이렇게 해서 어떤 Key가 중요한지 Weight를 만든다. 이후, 그 Weight와 Value를 곱하여 Output을 만든다.

<span style="color:#60a5fa">Self-Attention에서 Q, K, V는 **같은 입력 $X$에서 서로 다른 선형 변환**으로 만들어집니다. $Q = XW_Q$, $K = XW_K$, $V = XW_V$. 같은 입력에서 세 가지 역할을 분리하는 이유는, "무엇을 찾을지(Q)"와 "어떤 정보를 제공할지(K)"와 "실제 전달할 내용(V)"이 서로 다른 표현이어야 더 풍부한 관계를 학습할 수 있기 때문입니다.</span>

---

## Transformer 구조

Transformer의 Encoder에는 2가지 부분이 있다.

1. **Multi-Head Attention block**
2. **MLP / Feed-Forward block**

### Multi-Head Attention Block

![img11](/assets/lecture_img/transformer/img11.png)

Input → Norm → Multi-Head Attention → Residual Connection으로 원래 Input과 더한다.

![img12](/assets/lecture_img/transformer/img12.png)

Input → Layer Norm → Q, K, V → Attention(Q, K, V) → Output Projection → Residual Connection

- **LayerNorm**: 입력 벡터를 한 위치 기준으로 정규화해서 값 분포를 안정적으로 만드는 것
- **Output Projection**: Attention이 만든 결과를 다시 모델의 원래 표현 공간으로 섞어주는 선형 변환. Multi-Head Attention에서 여러 head 결과가 나오는데 이걸 하나로 합쳐 하나의 벡터로 정리한다.

### MLP Block

![img13](/assets/lecture_img/transformer/img13.png)

- 첫번째 projection: 차원 늘리기
- 두번째 projection: 원래 차원으로 돌리기
- MLP block: attention으로 섞인 정보를 각 위치별로 한 번 더 비선형적으로 가공

<span style="color:#60a5fa">Transformer에서 **Residual Connection**이 매우 중요합니다. 층이 깊어질수록 gradient가 사라지는 문제를 방지하기 위해, 각 sub-block의 출력에 입력을 그대로 더합니다. 이렇게 하면 최소한 항등 함수(identity)는 학습할 수 있으므로, 층을 추가해도 성능이 나빠지지 않습니다. 또한 **Pre-Norm**(LayerNorm을 attention/MLP 앞에 두는 방식)이 학습 안정성 면에서 유리해 현대 Transformer의 표준이 되었습니다.</span>

### 시퀀스 종류

- **Bidirectional** = 전체 문맥 다 봄
- **Causal** = 미래는 가리고 과거만 봄

---

## Type of Self-Attention

### 1. Masked Self-Attention

![img14](/assets/lecture_img/transformer/img14.png)

일반 self-attention은 각 위치가 모든 위치를 볼 수 있다. 즉 현재 단어도 보고, 이전도 보고, 미래도 보는데 **문장 생성에서는 이렇게 되면 답을 훔쳐보는 개념**이 된다. 따라서 현재 위치는 자기 자신과 과거까지만 볼 수 있게 하는 것이다.

즉, $x_2$는 $x_0, x_1, x_2$를 볼 수 있지만 $x_3, x_4$는 못 본다.

attention score에서 미래 값을 $-\infty$로 해서 softmax하면 가중치가 0이 되게 만든다.

### 2. Multi-Head Self-Attention

![img15](/assets/lecture_img/transformer/img15.png)

같은 self-attention을 여러 개 둬서 더 많은 표현 학습이 가능하다.

- 하나의 attention만 쓰면 한 종류의 관계만 보기 쉬움
- 여러 head를 쓰면 서로 다른 관계를 병렬로 볼 수 있어서 표현력이 더 커짐
- Conv도 여러 filter 쓰는 것처럼 비슷함

### 3. General Attention vs Self-Attention

![img16](/assets/lecture_img/transformer/img16.png)

- **General attention**: Q, K, V가 서로 다른 곳에서 올 수 있음
- **Self-attention**: Q, K, V가 **같은 입력**에서 옴. 이렇게 같은 입력에서 오기 때문에 모든 위치들이 직접 관계를 볼 수 있는 것이다.

<span style="color:#60a5fa">**Masked Attention**은 GPT 같은 autoregressive 모델의 핵심입니다. 학습 시에는 mask를 통해 미래 토큰을 가리지만, 추론 시에는 어차피 미래 토큰이 아직 생성되지 않았으므로 자연스럽게 causal 구조가 됩니다. **Multi-Head**에서 각 head가 실제로 서로 다른 패턴을 학습한다는 것은 실험적으로 확인되었습니다. 어떤 head는 문법적 관계를, 어떤 head는 의미적 유사성을, 어떤 head는 위치적 근접성을 주로 포착합니다.</span>
