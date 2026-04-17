---
title: "Statistical Image Models"
course: computer-vision
course_title: "Computer Vision"
order: 6
description: "자연 이미지의 통계적 특성부터 Wiener Filter, Wavelet Denoising, BM3D, DnCNN까지 이미지 복원의 흐름을 정리합니다."
mathjax: true
---

## Natural Image Statistics

![img1](/assets/lecture_img/denoising/img1.png)

자연 이미지(Natural Scenes)와 인공 이미지(Man-made Scenes)의 power spectrum을 분석하면, 자연 이미지의 주파수 스펙트럼은 다음과 같은 특성을 보인다.

$$
S(\nu) \propto \frac{1}{|\nu|^p}
$$

- 저주파 성분이 지배적이고, 고주파로 갈수록 에너지가 급격히 감소
- 자연 이미지는 **방향에 따라 비교적 균일**한 스펙트럼을 가짐
- 인공 이미지는 수평/수직 방향에 에너지가 집중되는 경향

<span style="color:#60a5fa">이 $1/|\nu|^p$ 법칙은 자연 이미지의 가장 기본적인 통계적 규칙성입니다. 직관적으로는 "자연 이미지에서 부드러운 영역(저주파)이 대부분이고, edge(고주파)는 상대적으로 드물다"는 의미입니다. 이 사전 지식이 denoising, super-resolution 등 이미지 복원 문제의 이론적 토대가 됩니다.</span>

---

## Noise Model

![img2](/assets/lecture_img/denoising/img2.png)

관측된 이미지 $f[n,m]$는 원본 이미지 $f_o[n,m]$에 노이즈 $\eta[n,m]$가 더해진 것으로 모델링한다.

$$
f[n,m] = f_o[n,m] + \eta[n,m]
$$

- $f_o$: 깨끗한 원본 이미지
- $\eta$: additive white Gaussian noise (AWGN)
- $f$: 관측된 noisy 이미지

주파수 영역에서 보면, 원본 이미지의 스펙트럼은 저주파에 에너지가 집중되지만 노이즈의 스펙트럼은 **모든 주파수에 균일하게 퍼져** 있다.

---

## Wiener Filter

![img3](/assets/lecture_img/denoising/img3.png)

Wiener Filter는 **주파수 영역에서 최적의 denoising**을 수행하는 선형 필터이다.

$$
\hat{F}_o(\nu) = \frac{|F_o(\nu)|^2}{|F_o(\nu)|^2 + \sigma_\eta^2} \cdot F(\nu)
$$

- 신호의 power가 큰 주파수 → 필터 값 ≈ 1 → 거의 그대로 통과
- 노이즈의 power가 지배적인 주파수 → 필터 값 ≈ 0 → 억제

즉, **SNR(Signal-to-Noise Ratio)이 높은 주파수는 살리고, 낮은 주파수는 죽이는** 방식이다.

<span style="color:#60a5fa">Wiener Filter는 MSE(Mean Squared Error)를 최소화하는 **이론적 최적 선형 필터**입니다. 하지만 실제로는 원본 이미지의 power spectrum $|F_o(\nu)|^2$를 알 수 없으므로, 자연 이미지의 $1/|\nu|^p$ 모델로 근사합니다. 또한 선형 필터이기 때문에 edge를 보존하면서 noise를 제거하는 데는 한계가 있습니다. 이를 극복하기 위해 비선형 방법들이 등장합니다.</span>

---

## Sparse Representation과 High-pass Filter

![img4](/assets/lecture_img/denoising/img4.png)

이미지에 high-pass filter(예: $[-1, 1]$)를 적용하면 edge 정보만 추출된다. 이 결과의 대부분의 값은 **0에 가까우며**, edge에 해당하는 소수의 값만 크다.

이것이 **sparse representation**의 핵심이다. 적절한 변환을 적용하면 자연 이미지의 계수(coefficient) 대부분이 0이고, 소수만 의미 있는 값을 가진다.

---

## Generalized Gaussian Distribution

![img5](/assets/lecture_img/denoising/img5.png)

High-pass filter를 통과한 이미지 계수의 분포는 **Generalized Gaussian Distribution**으로 모델링한다.

$$
p(\ell_h[n,m] = x) = \frac{\exp(-|x/s|^r)}{2(s/r)\Gamma(1/r)}
$$

파라미터 $r$에 따라 분포의 형태가 달라진다.

| $r$ | 분포 | 특성 |
|---|---|---|
| 0.5 | Super-Gaussian | 매우 뾰족, heavy tail |
| 1 | **Laplacian** | 뾰족한 분포 |
| 2 | **Gaussian** | 종 모양 |
| $\infty$ | Uniform | 균일 분포 |

자연 이미지의 wavelet/gradient 계수는 $r \approx 0.5 \sim 1$ 정도로, **Laplacian에 가까운 분포**를 따른다. 즉, 대부분의 계수가 0 근처에 몰려 있고 가끔 큰 값(edge)이 나타나는 **sparse** 한 분포이다.

<span style="color:#60a5fa">이 분포가 denoising에서 중요한 이유는 **prior(사전 확률)**로 사용되기 때문입니다. Gaussian noise의 likelihood와 Laplacian prior를 결합하면 Bayesian 추정이 가능해집니다. Laplacian prior는 "대부분의 계수는 0이어야 한다"는 sparsity 가정을 수학적으로 표현한 것이고, 이는 L1 regularization과 동일합니다.</span>

---

## Bayesian Wavelet Denoising

![img6](/assets/lecture_img/denoising/img6.png)

Bayes 정리를 이용해 denoising을 확률적으로 수행한다.

$$
P(x|y) \propto P(y|x) \cdot P(x)
$$

- $y = x + \eta$: noise-corrupted observation
- $\eta \sim \mathcal{N}(0, \sigma^2)$: Gaussian noise
- $x$: bandpass된 원본 이미지 값

각 wavelet 계수에 대해:

- **Gaussian likelihood** $P(y|x)$: 관측 모델 (노이즈 특성)
- **Laplacian prior** $P(x)$: 자연 이미지의 sparse 특성

이 둘을 곱하면 **posterior** $P(x|y)$를 얻고, 그 peak(MAP estimate)가 denoised 값이 된다.

- 관측 계수가 작으면 → noise일 가능성 높음 → **0으로 shrink**
- 관측 계수가 크면 → 실제 edge일 가능성 높음 → **거의 그대로 유지**

<span style="color:#60a5fa">이 방식을 **wavelet shrinkage**라고 부르며, Donoho와 Johnstone이 제안한 soft/hard thresholding이 대표적입니다. Threshold $\tau$를 기준으로 $|y| < \tau$이면 0으로, $|y| \geq \tau$이면 $y - \text{sign}(y) \cdot \tau$로 줄이는 것이 soft thresholding입니다. 이론적으로 minimax optimal에 가까운 성능을 보이며, 이후 BM3D 같은 방법의 이론적 기반이 됩니다.</span>

---

## Non-local Means와 Self-Similarity

![img7](/assets/lecture_img/denoising/img7.png)

자연 이미지에는 **self-similarity**가 존재한다. 이미지 내에서 비슷한 패치가 여러 곳에 반복적으로 나타나며, 이 성질을 활용하면 더 효과적인 denoising이 가능하다.

Non-local Means는 현재 픽셀 주변뿐 아니라, 이미지 전체에서 **비슷한 패치를 찾아** 가중 평균을 내는 방식이다.

<span style="color:#60a5fa">Non-local Means의 아이디어는 "비슷한 패치는 같은 원본에서 왔을 것이므로, 평균을 내면 noise가 상쇄된다"는 것입니다. 이는 local filter(Gaussian blur 등)와 근본적으로 다른 접근입니다. Local filter는 가까운 픽셀끼리만 평균을 내므로 edge가 뭉개지지만, non-local 방식은 **의미적으로 비슷한 픽셀**끼리 평균을 내므로 edge를 보존할 수 있습니다.</span>

---

## BM3D

![img8](/assets/lecture_img/denoising/img8.png)

**BM3D(Block-Matching and 3D Filtering)** 는 전통적 denoising 방법 중 가장 강력한 성능을 보이는 알고리즘이다. Self-similarity를 적극 활용한다.

### 동작 과정

1. **Block Matching**: 이미지에서 비슷한 패치들을 찾아 그룹으로 묶음
2. **3D Transform**: 묶인 패치 그룹을 3D 배열로 쌓고, 3D 변환(wavelet 등)을 적용
3. **Shrinkage**: 변환 계수에 thresholding을 적용해 noise 제거
4. **Inverse Transform**: 역변환 후 원래 위치로 돌려놓고 가중 평균

<span style="color:#60a5fa">BM3D가 강력한 이유는 **두 가지 sparsity를 동시에 활용**하기 때문입니다. 하나는 2D 변환(DCT/wavelet)에 의한 각 패치 내부의 sparsity이고, 다른 하나는 비슷한 패치들을 쌓아 3번째 축으로 변환했을 때의 sparsity입니다. 비슷한 패치끼리는 3번째 축에서 거의 동일하므로, 변환하면 에너지가 하나의 계수에 집중됩니다. 이 알고리즘은 딥러닝 이전 시대의 최강 denoiser로, 오랫동안 벤치마크의 기준이 되었습니다.</span>

---

## Deep Learning-based Denoising: DnCNN

![img9](/assets/lecture_img/denoising/img9.png)

**DnCNN**은 딥러닝으로 denoising을 수행하는 대표적인 모델이다.

핵심 아이디어는 깨끗한 이미지를 직접 예측하는 것이 아니라, **noise를 예측(residual learning)** 하는 것이다.

$$
\hat{\eta} = f_\theta(y), \quad \hat{x} = y - \hat{\eta}
$$

- 입력: noisy image $y$
- 네트워크 출력: 예측된 noise $\hat{\eta}$
- 복원 이미지: $\hat{x} = y - \hat{\eta}$

구조는 여러 층의 Conv + BatchNorm + ReLU로 구성된 간단한 CNN이다.

<span style="color:#60a5fa">Residual learning이 직접 clean image를 예측하는 것보다 좋은 이유는, noise가 이미지 자체보다 훨씬 **구조가 단순**하기 때문입니다. 네트워크가 복잡한 이미지 구조를 학습하는 대신 상대적으로 단순한 noise 패턴만 학습하면 되므로 수렴이 빠르고 성능이 좋습니다. DnCNN은 BM3D를 능가하는 성능을 보이면서도, 한 번의 forward pass로 처리할 수 있어 속도도 훨씬 빠릅니다. 이후 FFDNet, DRUNet 등 다양한 발전된 모델들이 등장했습니다.</span>

---

## 전통 방법 vs 딥러닝 요약

| | 전통 방법 (Wiener, BM3D 등) | 딥러닝 (DnCNN 등) |
|---|---|---|
| Prior/가정 | 수학적으로 명시 (sparsity, self-similarity) | 데이터로부터 암묵적으로 학습 |
| 해석 가능성 | 높음 | 낮음 |
| 속도 | BM3D는 느림 | Forward pass 한 번, 빠름 |
| 성능 | 오래 최강이었으나 한계 | BM3D를 넘어섬 |
| 일반화 | noise 모델 가정에 의존 | 다양한 noise에 학습 가능 |
