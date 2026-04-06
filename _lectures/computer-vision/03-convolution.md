---
title: "Convolution Operation"
course: computer-vision
course_title: "Computer Vision"
order: 3
description: "이미지 처리의 핵심 연산인 컨볼루션의 수학적 정의와 동작 원리를 설명합니다."
mathjax: true
---

## 왜 컨볼루션인가?

일반 Fully Connected(FC) 레이어는 이미지 전체 픽셀을 한꺼번에 연결합니다. 예를 들어 $224 \times 224 \times 3$ 이미지는 약 **150,000개** 입력이고, 첫 FC 레이어에서만 수억 개 파라미터가 생깁니다.

**컨볼루션**은 두 가지 핵심 아이디어로 이를 해결합니다:

1. **지역성(Locality)**: 인접 픽셀끼리만 연결 (작은 필터)
2. **가중치 공유(Weight Sharing)**: 같은 필터를 이미지 전체에 슬라이딩

---

## 2D Convolution 정의

입력 $I$에 필터(커널) $K$를 적용해 출력 $O$를 얻습니다.

$$
O[i, j] = \sum_{m} \sum_{n} I[i+m,\; j+n] \cdot K[m, n]
$$

- 필터 크기: 보통 $3 \times 3$, $5 \times 5$
- 필터 값은 학습으로 결정됨

---

## 슬라이딩 과정

```
Input (5×5):          Filter (3×3):
1  2  3  4  5         1  0  -1
6  7  8  9  10        1  0  -1
11 12 13 14 15        1  0  -1
16 17 18 19 20
21 22 23 24 25

→ 필터를 왼쪽에서 오른쪽, 위에서 아래로 이동하며 내적 계산
```

출력 크기(패딩 없을 때):

$$
O_H = I_H - K_H + 1, \quad O_W = I_W - K_W + 1
$$

---

## Padding & Stride

### Padding
출력 크기를 입력과 동일하게 유지하기 위해 가장자리에 0을 채웁니다.

$$
\text{Same padding: } p = \lfloor K/2 \rfloor
$$

### Stride
필터가 한 번에 이동하는 픽셀 수. Stride $s > 1$이면 출력이 작아집니다.

$$
O_H = \left\lfloor \frac{I_H - K_H + 2p}{s} \right\rfloor + 1
$$

---

## 필터가 학습하는 것

| 필터 종류 | 검출되는 특징 |
|---|---|
| 수평 엣지 필터 | 수평 경계선 |
| 수직 엣지 필터 | 수직 경계선 |
| 가우시안 필터 | 블러(잡음 제거) |
| Laplacian | 경계 전반 |

딥러닝에서는 **필터 값을 직접 설계하지 않고** 역전파로 자동 학습합니다.

---

## 다음 강의 예고

다음 시간에는 **Pooling** 연산과 CNN 전체 구조(Conv → Pool → FC)를 살펴봅니다.
