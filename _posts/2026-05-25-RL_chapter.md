---
title: "16강 PPO"
description: "PPO"
author: KEYBEOMM
date: 2026-05-23 01:00:00 +0900
categories: [RL]
tags: [RL, PPO]
math: true
---

# 16주차 PPO (Proximal Policy Optimization) 강의 정리본

> **전사본 품질 경고**: 이번 녹음 전사는 품질이 현저히 낮음. 중간중간 "K K K K..." 반복 구간, 의미없는 영문 삽입 등 내용 손실 구간이 많음. 교수님의 실제 발화 표현을 충분히 살리지 못한 부분이 있으며, 해당 구간은 강의자료로 보충함. 보충 부분은 `[보충: ...]`으로 표시.

---

## 0. 강의 한눈에 보기

**교수님의 큰 그림 (프레이밍)**

강의 시작에서 교수님은 바로 이 두 알고리즘의 위상을 정의했다.

> "Model-free 강화학습에서 가장 중요한 두 개를 뽑으라고 하면, PPO(on-policy)와 SAC(off-policy)예요. OpenAI 쪽에서 나오는 연구들은 주로 PPO를 쓰고, UC Berkeley 쪽에서는 SAC로 가던 것 같아요. 진영이 나뉘어져 있었는데, 해를 넘겨서 어느 정도 그런 구분이 없어지는 추세가 됐으면 좋겠습니다."

이번 강의는 **지난 시간 TRPO 복습 → PPO (Clipped Objective) → PPO-v1 (Adaptive KL) → GRPO** 순으로 진행.

**다룬 개념 목록 및 중요도**

| 개념 | 중요도 |
|------|--------|
| 폴리시 업데이트 크기 문제 / RL 연쇄 붕괴 | ★★★ |
| TRPO 복습 (surrogate objective, constraint/penalty form) | ★★ |
| PPO: Clipped Objective ($J_{CLIP}$) | ★★★ |
| PPO 알고리즘 (K-step minibatch SGD) | ★★ |
| PPO-v1: Adaptive KL Penalty | ★ |
| GRPO: Group Relative Policy Optimization | ★★★ |
| PPO vs GRPO 비교 | ★★ |

**지난 강의와의 연결**: TRPO (15주차)에서 직접 이어짐. 교수님이 "TRPO를 안 가르치면 PPO 설명이 애매해진다, 어쩔 수 없이 TRPO를 넣었다"고 직접 언급.

---

## 1. 폴리시 업데이트 크기 문제 — RL 연쇄 붕괴 ★★★

> **★★★** — 강의 전체 도입부 논리. 교수님이 도파민 비유·안개 낀 절벽 비유·일반 NN과의 대조 등 3가지 비유를 동원해 반복 설명. 강의자료 p.2 노란 형광펜 강조.

**교수님의 설명 흐름**

어드밴테이지의 의미부터 다시 짚는다. 어떤 상태에서 특정 액션의 어드밴테이지가 양수라는 것은, 그 상태의 평균적인 액션들보다 이 액션의 가치(Q-V)가 높다는 뜻이다. 그러면 당연히 그 액션을 더 많이 하도록 폴리시를 업데이트하고, 음수면 덜 하도록 낮춰야 한다.

여기서 교수님이 핵심 질문을 던진다: **"얼마만큼 바꿔도 되느냐?"**

논리적으로는 좋은 액션이니까 확률 1로, 나쁜 액션이니까 확률 0으로 만들면 objective가 제일 높아지지 않느냐 — 하지만 이렇게 하면 학습이 망가진다.

**핵심 정의** (강의자료 p.2)

- 폴리시 업데이트가 너무 작으면 → **학습이 느림**
- 폴리시 업데이트가 너무 크면 → **Performance collapse (모델 붕괴)**

**교수님이 쓴 비유·예시·표현**

**도파민 비유**:
> "어드밴테이지가 도파민이에요. 도파민을 주는 어드밴테이지가 있다라고 하면, 여러분들은 그 행동을 더 많이 하려고 하잖아요. 네거티브는 고통이에요. 고통을 주는 액션은 안 하려고 하는 거죠."

**일반 NN vs RL 연쇄 붕괴 대조** (전사본에서 거의 완전히 복원 가능한 핵심 구간):
> "ResNet으로 ImageNet을 학습시킬 때, 이상한 데이터셋이 들어와서 파라미터가 잠깐 망가질 수도 있어요. 근데 그 다음에 들어오는 데이터셋들이 다시 학습해서 돌아올 수 있잖아요. 근데 강화학습은 그게 안 되는 이유가 뭐냐면, 우리가 학습한 걸로 또 데이터를 만들어서 학습을 하기 때문에 — 한 번 망가지면 망가진 모델로 만든 데이터도 제대로 된 데이터가 아니고, 그 데이터로 또 학습을 하면 또 망가지고, 이게 반복이 되기 때문에 강화학습은 한 번 모델이 망가지면 계속 망가진다는 거예요. 그게 일반 지도학습과 차이가 있는 거예요."

**안개 낀 절벽 비유** (Trust Region의 직관):
> "안개가 끼었는데 낭떠러지가 있어요. 꼭대기로 가기는 가야 되는데 방향은 알 것 같아요. 근데 한 발을 너무 크게 내디디면 낭떠러지로 떨어질 수도 있잖아요. 그러면 어디까지 가는 게 안전하냐, 한 걸음을 몇 센치로 걷는 것이 낭떠러지에 떨어지지 않고 계속 올바르게 학습할 수 있느냐 — 이 질문에 대한 대답이 Trust Region Policy Optimization이에요."

**강조 포인트**

RL의 연쇄 붕괴가 일반 지도학습과 다른 근본 이유: **데이터 생성 자체가 현재 폴리시에 의존**한다. 지도학습은 외부에서 고정된 데이터가 주어지지만, RL은 폴리시가 망가지면 수집 데이터도 동시에 망가진다.

---

## 2. TRPO 복습 ★★

> **★★** — 지난 시간 내용이라 복습 포지션이지만, PPO를 이해하는 필수 기반. 교수님이 TRPO의 문제점을 설명하는 부분이 PPO 도입의 직접적인 동기.

**교수님의 설명 흐름**

TRPO는 old policy의 샘플을 재활용하되, importance sampling 비율로 보정해서 새로운 폴리시의 objective를 추정한다.

**핵심 정의 / 수식** (강의자료 p.3)

$$J(\theta) = \sum_s\sum_a \pi_{\theta_{OLD}}(s,a) \frac{\pi_\theta(a|s)}{\pi_{\theta_{OLD}}(a|s)} A_{\theta_{OLD}}(s,a) = \mathbb{E}_{\pi_{\theta_{OLD}}}\left[\frac{\pi_\theta(a|s)}{\pi_{\theta_{OLD}}(a|s)} A_{\theta_{OLD}}(s,a)\right]$$

State distribution $\pi_\theta(s) \neq \pi_{\theta_{OLD}}(s)$이지만, **두 폴리시가 크게 다르지 않다는 가정 하에** state distribution을 old policy로 근사.

**Constraint Form** (강의자료 p.4):
$$\text{maximize}_\theta \; \hat{\mathbb{E}}_t\left[\frac{\pi_\theta(a_t|s_t)}{\pi_{\theta_{old}}(a_t|s_t)}\hat{A}_t\right], \quad \text{subject to} \; \hat{\mathbb{E}}_t\left[\text{KL}\left[\pi_{\theta_{old}}(\cdot|s_t), \pi_\theta(\cdot|s_t)\right]\right] \leq \delta$$

**Penalized Form**:
$$\text{maximize}_\theta \; \hat{\mathbb{E}}_t\left[\frac{\pi_\theta(a_t|s_t)}{\pi_{\theta_{old}}(a_t|s_t)}\hat{A}_t - \beta \, \text{KL}\left[\pi_{\theta_{old}}(\cdot|s_t), \pi_\theta(\cdot|s_t)\right]\right]$$

여기서 $r_t(\theta) = \frac{\pi_\theta(a_t|s_t)}{\pi_{\theta_{old}}(a_t|s_t)}$ 로 정의하면 $r(\theta_{old}) = 1$.

**TRPO의 문제점** (강의자료 p.4, 교수님 설명):
- Hard to train: 2차 최적화 필요 (Fisher matrix, conjugate gradient, line search)
- 단일 $\beta$ 값이 **모든 환경·에피소드에서 최적**이기 어려움 → β를 잘 고르는 것 자체가 어려운 문제

---

## 3. PPO: Clipped Objective ★★★

> **★★★** — 교수님이 "이것이 PPO의 전문입니다"를 **세 번 반복**. "이것만 이해하시면 됩니다. 1분 동안 이해해 보세요"라고 직접 발화. 강의 시간의 가장 많은 비중을 차지.

**교수님의 설명 흐름**

TRPO는 복잡하다. PPO는 **같은 trust region 직관을 clip 함수 하나로 대체**한다. KL constraint도, Fisher matrix도, line search도 없이.

**핵심 정의 / 수식** (강의자료 p.6-9)

probability ratio:
$$r(\theta) = \frac{\pi_\theta(a|s)}{\pi_{\theta_{old}}(a|s)}, \quad r(\theta_{old}) = 1$$

clip function (강의자료 p.8):
$$\text{clip}(r(\theta), 1-\epsilon, 1+\epsilon) = \begin{cases} 1+\epsilon & \text{if } r(\theta) \geq 1+\epsilon \\ 1-\epsilon & \text{if } r(\theta) \leq 1-\epsilon \\ r(\theta) & \text{otherwise} \end{cases}$$

**PPO Clipped Objective** (강의자료 p.9):
$$J_{CLIP}(\theta) = \hat{\mathbb{E}}_t\left[\min\left(r_t(\theta)\hat{A}_{\theta_{old}}(s,a),\ \text{clip}(r(\theta), 1-\epsilon, 1+\epsilon)\hat{A}_{\theta_{old}}(s,a)\right)\right]$$

**교수님이 쓴 비유·표현**

> "r이 1보다 크다는 것은 바뀐 액션의 확률이 더 커졌다는 거고, 1보다 작으면 더 줄었다는 거예요."

> "TRPO는 trust region까지 가는 방법을 정확하게 계산해서 딱 한 번에 가는 거예요. PPO는 조금씩 조금씩 걸어가다가 그 경계에 다다르는 방식이에요."

---

### Case 1: $\hat{A}_{old} > 0$ — 좋은 액션, 확률 올리기 (강의자료 p.9-11)

- 이 액션이 해당 상태 평균보다 좋다 → r을 키울수록 J가 커짐
- clip에 의해 r은 최대 $1+\epsilon$까지만 증가
- r > $1+\epsilon$ → gradient = 0 → **"ceiling" 효과**

**Walk-through 예시** ($\pi_{old}=0.5$, $\hat{A}=+2$, $\epsilon=0.2$) — 강의자료 p.11:

| $\pi_{new}$ | $r$ | $L^{CLIP}$ | 비고 |
|---|---|---|---|
| 0.4 | 0.8 | 1.6 | 반대 방향 |
| 0.5 | 1.0 | 2.0 | 시작점 |
| 0.6 | 1.2 | 2.4 | **경계** |
| 0.7 | 1.4 | 2.4 | CLIPPED — grad = 0 |

> r > 1+ε이 되면 loss가 flat해진다. optimizer가 $\pi_{new}$를 더 올릴 gradient를 받지 못함 → ceiling.

---

### Case 2: $\hat{A}_{old} < 0$ — 나쁜 액션, 확률 내리기 (강의자료 p.12-14)

- 이 액션이 평균보다 나쁘다 → r을 0에 가깝게 줄일수록 J가 커짐 (음수 × 작은 양수 = 덜 음수)
- clip에 의해 r은 최소 $1-\epsilon$까지만 감소
- r < $1-\epsilon$ → gradient = 0 → **"floor" 효과**

**교수님의 비유** (전사본에서 복원):
> "어드밴테이지가 네거티브니까 부끄러운 거예요. 부끄러우니까 숨기려 할 거잖아요. 그러면 r 값을 0에 가깝게 만들면 J가 커지는 거죠."

교수님이 "이 부분이 포지티브 케이스보다 좀 헷갈리실 수 있어요"라고 직접 경고.

**Walk-through 예시** ($\pi_{old}=0.5$, $\hat{A}=-2$, $\epsilon=0.2$) — 강의자료 p.14:

| $\pi_{new}$ | $r$ | $L^{CLIP}$ | 비고 |
|---|---|---|---|
| 0.6 | 1.2 | -2.4 | 반대 방향 |
| 0.5 | 1.0 | -2.0 | 시작점 |
| 0.4 | 0.8 | -1.6 | **경계** |
| 0.3 | 0.6 | -1.6 | CLIPPED — grad = 0 |

> r < 1-ε이 되면 loss가 flat해진다. optimizer가 $\pi_{new}$를 더 낮출 gradient를 받지 못함 → floor.

---

**강조 포인트 & 중요도 근거**

- "이것이 PPO의 전문입니다"를 세 번 반복 → 명시적 최고 강조
- $\epsilon \approx 0.2$ 하나로 대부분의 환경에서 잘 작동 → 강의자료 p.18 실험 결과 (Clipping $\epsilon=0.2$ avg. 0.82로 최고)
- KL constraint 없음, Fisher matrix 없음, line search 없음 → 단순함
- First-order 최적화만으로 trust region 효과 달성

**주의·함정**

- $A < 0$ 케이스가 직관적으로 헷갈림: A가 음수일 때 J를 높이려면 r을 **줄여야** 한다. r과 A 부호 관계를 혼동하지 말 것.
- clip은 **휴리스틱** — TRPO처럼 단조 개선 수학적 보장이 없다 (강의자료 p.20).
- 교수님이 PPO 논문에는 버전 1(KL)과 버전 2(Clip) 두 가지가 있는데, 실험 결과 오히려 KL 안 쓴 버전이 성능이 더 좋았다고 언급.

---

## 4. PPO 알고리즘 ★★

> **★★** — 알고리즘 자체는 Actor-Critic의 확장. 교수님이 K-step iteration의 의미와 on-policy이지만 데이터 효율이 높은 이유를 강조.

**핵심 정의 / 수식** (강의자료 p.15)

**Algorithm 5 — PPO with Clipped Objective**:

```
Input: θ₀, ε
for k = 0, 1, 2, ... do
  1. Rollout: π_k로 partial trajectory D_k 수집
  2. Advantage 추정: 어떤 방법이든 Â^{π_k}_t 계산  ← value network 필요!
  3. K steps of minibatch SGD (Adam)으로 policy update:
     θ_{k+1} = argmax_θ L^{CLIP}_{θ_k}(θ)
end for
```

$$\mathcal{L}^{CLIP}_{\theta_k}(\theta) = \mathbb{E}_{\tau \sim \pi_k}\left[\sum_{t=0}^T \min\left(r_t(\theta)\hat{A}_t^{\pi_k},\ \text{clip}\left(r_t(\theta), 1-\epsilon, 1+\epsilon\right)\hat{A}_t^{\pi_k}\right)\right]$$

**교수님의 설명**

> "하나의 trajectory(SARS)가 들어오면, 그 SARS에 대한 현재 폴리시의 확률과 어드밴테이지가 있잖아요. 그걸로 폴리시를 K번 반복해서 업데이트합니다. 업데이트할 때마다 $r = \pi_{new}/\pi_{old}$가 계속 변해요. 두 번째 업데이트 만에 1+ε에 도달할 수도 있어요. 그때부터는 gradient가 0이 돼서 더 이상 안 바뀌죠. 하지만 다섯 번 하기로 했으면 다섯 번을 합니다. 거기서 더 가보는 게 좋냐는 방식으로 사용을 합니다."

**TRPO와의 차이**:
- TRPO: trust region까지 정확히 계산해서 **한 번** 업데이트
- PPO: 같은 데이터로 **K번** minibatch SGD → 데이터 효율↑, 구현 간단↑

**중요 포인트** (강의자료 p.23 "The PPO detail you missed"):
Advantage 추정에 **Critic (value network)이 필요**하다. PPO = Actor(policy) + Critic(value). 이것이 나중에 GRPO와의 핵심 차이점.

**PPO Key Virtues** (강의자료 p.20):
- **SIMPLICITY**: First-order 최적화, ~50줄 코드
- **ROBUSTNESS**: 하이퍼파라미터에 둔감
- **EFFICIENCY**: 한 rollout으로 K epoch 업데이트 (TRPO 대비)
- **SCALABILITY**: 병렬화 용이
- **GENERALITY**: 이산·연속 action space 모두 가능

**PPO의 한계** (강의자료 p.20):
- On-policy → off-policy(SAC, TD3) 대비 sample-inefficient
- Clipping은 heuristic — TRPO의 단조 개선 보장 없음
- LLM 학습에서 비용 문제 → GRPO 등장 배경

---

## 5. Appendix: Adaptive KL Penalty (PPO-v1) ★

> **★** — 강의자료에 "Appendix"로 분류. 교수님이 "버전 1은 사실 여기 패스합니다"처럼 빠르게 넘김. 논문 원본의 두 버전을 아는 맥락용.

**핵심 정의 / 수식** (강의자료 p.16-17)

$$L^{KLPEN}(\theta) = \hat{\mathbb{E}}_t\left[\frac{\pi_\theta(a_t|s_t)}{\pi_{\theta_{old}}(a_t|s_t)}\hat{A}_t - \beta \, \text{KL}\left[\pi_{\theta_{old}}(\cdot|s_t), \pi_\theta(\cdot|s_t)\right]\right]$$

$\beta$ 적응 규칙:
- $d = \text{KL}[\pi_{\theta_{old}}, \pi_\theta]$ 계산
- If $d < d_{targ}/1.5$: $\beta \leftarrow \beta/2$ (KL이 목표보다 작으면 β를 줄임)
- If $d > d_{targ} \times 1.5$: $\beta \leftarrow \beta \times 2$ (KL이 너무 크면 β를 늘림)

**교수님의 설명**:
> "PPO 논문에는 두 버전이 있어요. 버전 1이 KL divergence를 쓴 거고, 버전 2가 clip이에요. 근데 실험 결과 오히려 KL 안 쓴 것이 성능이 좋게 나왔어요. 그래서 우리가 PPO라고 하면 clip 버전을 말하는 거예요."

**실험 결과 요약** (강의자료 p.18, Continuous Control Benchmark):

| 방법 | avg. normalized score |
|------|----------------------|
| No clipping or penalty | -0.39 |
| **Clipping, ε=0.2** | **0.82** |
| Adaptive KL ($d_{targ}=0.01$) | 0.74 |
| Fixed KL, β=3 | 0.72 |

→ Clipping $\epsilon=0.2$가 전체 최고 성능.

---

## 6. GRPO: Group Relative Policy Optimization ★★★

> **★★★** — 강의자료 p.21-31로 분량 최대. DeepSeek-R1, ChatGPT 등 현재 LLM RL의 핵심 알고리즘. 교수님이 "여러분들이 더 알아봐야 할 것"이라고 언급.

**교수님의 설명 흐름**

PPO는 좋은 알고리즘이지만 LLM에 적용할 때 메모리 문제가 있다. 강의자료 p.22에 명시된 맥락:

| 연도 | 사건 |
|------|------|
| 2017 | PPO 등장 (Schulman et al.) |
| 2022 | ChatGPT — PPO로 RLHF → 대중화 |
| 2023 | RLHF 확산 — 모두 PPO, 하지만 7B×4=28B 파라미터 |
| 2024 | DeepSeekMath — GRPO 제안 → 메모리 절약 + 좋은 성능 |
| 2025 | DeepSeek-R1 — GRPO + pure RL → o1급 성능을 저비용으로 |

> "7B policy + 7B value + 7B reward + 7B reference = 28B 파라미터를 메모리에 올려야 해요. 'RL for math reasoning is too expensive!' 이게 DeepSeekMath 팀의 문제의식이에요."

**핵심 아이디어**: Critic (value network)을 제거하고, 같은 입력에서 G개의 출력을 생성한 다음, **그 보상들의 평균과 표준편차로 advantage를 직접 계산**.

**핵심 정의 / 수식** (강의자료 p.21, 27)

Group-relative advantage:
$$\hat{A}_i = \frac{r_i - \text{mean}(\{r_1,...,r_G\})}{\text{std}(\{r_1,...,r_G\})}$$

Importance ratio:
$$\rho_{i,t}(\theta) = \frac{\pi_\theta(o_{i,t} \mid q, o_{i,<t})}{\pi_{\theta_{old}}(o_{i,t} \mid q, o_{i,<t})}$$

**GRPO Objective** (강의자료 p.27):
$$\mathcal{J}_{GRPO}(\theta) = \frac{1}{G}\sum_{i=1}^G\frac{1}{|o_i|}\sum_{t=1}^{|o_i|}\left\{\min\left[\rho_{i,t}\hat{A}_{i,t},\ \text{clip}(\rho_{i,t}, 1-\varepsilon, 1+\varepsilon)\hat{A}_{i,t}\right] - \beta D_{KL}[\pi_\theta \| \pi_{ref}]\right\}$$

**PPO와 GRPO의 KL 기준이 다르다** — 중요 포인트:
- PPO: KL($\pi_{old} \| \pi_{new}$) — 이전 업데이트 전과 비교
- GRPO: KL($\pi_\theta \| \pi_{ref}$) — **학습 시작 시 초기 모델**과 비교 → catastrophic forgetting 방지

**교수님의 설명** (전사본에서 복원):
> "reference policy($\pi_{ref}$)라는 게 뭐냐면, 매 업데이트 전의 old policy가 아니라, 학습을 시작할 때의 모델이에요. 어떤 수학문제에 대해서 GRPO 방식으로 수학 실력을 올릴 때, 기존에 갖고 있던 다른 문제들도 다 바보가 돼버려요 — 파라미터가 바뀌면서. 그걸 막기 위해서 기존 능력을 유지하면서 수학 실력을 올린다는 거죠. 최대한 원래 리더블이 되면서 발전한다는 거죠."

---

### GRPO의 KL Divergence Estimator

**Naive estimator의 문제** (강의자료 p.28):
$$\hat{D}^{k1}_{KL} = \log\frac{\pi_\theta(x)}{\pi_{ref}(x)}$$
- 개별 샘플에서 **음수**가 될 수 있음 (KL은 항상 ≥ 0이어야 함)
- **분산이 높음** → gradient 신호가 노이지

**Schulman's Estimator (k3)** (강의자료 p.29):
$$\hat{D}^{k3}_{KL} = \frac{\pi_{ref}(x)}{\pi_\theta(x)} - \log\frac{\pi_{ref}(x)}{\pi_\theta(x)} - 1$$

$r = \pi_{ref}/\pi_\theta$로 놓으면 이 식이 $r - \log r - 1$.

- **Unbiased 증명**: $\mathbb{E}_{x \sim \pi_\theta}[r] = \sum_x \pi_\theta(x) \cdot \frac{\pi_{ref}(x)}{\pi_\theta(x)} = \sum_x \pi_{ref}(x) = 1$, $\mathbb{E}[\log r] = -D_{KL}$이므로 $E[r] - E[\log r] - 1 = 1 + D_{KL} - 1 = D_{KL}$ ✓
- **항상 ≥ 0**: $r - \log r - 1 \geq 0$ (r=1에서 global minimum = 0) ✓

---

### GRPO가 작동하는 4가지 조건 (강의자료 p.25)

1. 같은 입력에서 **G개의 출력을 쉽게 생성** 가능해야 함
   - LLM: "2x+4=10" → 16개 응답을 한 batch에서 생성
   - 로봇: 같은 초기 상태에서 16번 시뮬레이션 (비쌈)
2. 보상이 trajectory **끝에 한 번** 주어져야 함 (sparse reward)
3. 같은 입력에서의 출력들이 **의미 있게 비교** 가능해야 함
   - 수학: 맞다/틀리다로 직접 비교 가능
4. Value network가 policy만큼 **비싸야** GRPO의 절약이 실질적 이득

**교수님의 설명**:
> "수학, 코딩 같은 건 답이 맞다/틀리다가 있어요. 코딩은 실행해보면 되고, 수학은 검산하면 되잖아요. 근데 글쓰기나 preference 같은 건 맞다/틀리다 자체가 애매해요. 그런 건 GRPO보다 PPO가 더 적합해요."

**GRPO가 강한 도메인 vs 그렇지 않은 도메인** (강의자료 p.26):

| GRPO가 강한 영역 | PPO가 더 적합한 영역 |
|---|---|
| 수학 추론 (DeepSeekMath: 46.8→51.7%) | 일반 RLHF (preference learning) |
| 코딩 (unit test 검증) | 글쓰기 품질, 안전성 |
| Chain-of-thought 추론 (DeepSeek-R1) | 단답형 (group signal 약함) |

**GRPO의 강점과 한계** (강의자료 p.31):

| 강점 | 한계 |
|------|------|
| Critic 없음 → 메모리 40-60% 절약 | G개 출력 생성 비용 (보통 8-64개) |
| PPO clipping 재사용 → 구현 간단 | 토큰별 credit이 균일 (GAE보다 coarse) |
| Group normalization → 안정적 advantage | G가 작으면 mean/std가 불안정 |
| Bootstrapping bias 없음 | Dense-reward에서 PPO가 더 좋을 수 있음 |

---

## 7. PPO vs GRPO 비교 ★★

> **★★** — 강의자료 p.24에 명시적 비교. 두 알고리즘의 설계 철학 차이를 이해하는 핵심.

| 항목 | PPO | GRPO |
|------|-----|------|
| 네트워크 수 | Policy + Value (2개) | Policy만 (1개) |
| Advantage 추정 | Critic (GAE) | Group mean/std |
| KL 기준 | $\pi_{old}$ (직전 업데이트) | $\pi_{ref}$ (학습 초기 모델) |
| 메모리 | 상대적으로 큼 | 40-60% 절약 |
| 구현 복잡도 | Critic 별도 설계/튜닝 필요 | 상대적으로 간단 |
| 적합 도메인 | Dense reward, general RLHF | Sparse/verifiable reward |

**DeepSeekMath 실험 결과** (강의자료 p.30):

| Benchmark | SFT | GRPO (RL) |
|-----------|-----|-----------|
| GSM8K | 83% | 88% |
| MATH | 47% | 52% |
| CMath | 85% | 89% |
| MGSM-zh | 73% | 80% |

---

## 마지막 점검

### 전사 오류 교정 (명백한 것)

| 전사본 표기 | 실제 의미 |
|------------|----------|
| "TIPO" | TRPO (Trust Region Policy Optimization) |
| "GIPO" | GRPO (Group Relative Policy Optimization) |
| "폼폴릿" | on-policy |
| "KL-Diversion" | KL-Divergence |
| "어드밴트 행위" | advantage |
| "SARS" | (s, a, r, s') — 강화학습 transition tuple |

### 내용 손실 구간 (전사 불명확, 추정 불가)

- **"K K K K K..." 반복 구간**: TRPO constraint form 설명 후반부. 강의자료 p.4로 보충.
- **"Fly", "Coco", "Let's dog" 등 의미없는 삽입 구간**: TRPO→PPO 전환 설명 부분. 내용 복원 불가.
- **GRPO KL-divergence 도입부 설명 일부**: "이 사람이 블록을 사용해서..." 이후 구간. 강의자료 p.27-29로 보충.
- **PPO K-step 반복 이후 rollout 설명 구간**: 일부 복원, 일부 손실.

### 추정으로 채운 부분

- TRPO Penalized Form과 $\delta$ 설정 어려움 설명 → 강의자료 기반 [보충]
- PPO-v1 알고리즘 세부 → 강의자료 p.17 기반 [보충]
- Schulman's estimator 수식 증명 → 강의자료 p.29 기반 [보충]

### 강의자료 표기 오류로 의심되는 부분

- 강의자료 p.13 슬라이드 헤더가 "When $\hat{A}_{old} > 0$"인데, 그래프 내용은 $A < 0$ 케이스임. 강의자료 자체의 표기 오류로 판단 (맥락상 $A < 0$ 케이스 설명 슬라이드가 맞음).

### 교수님이 예고한 다음 내용

- **SAC (Soft Actor-Critic)** — PPO의 off-policy 상대방. 교수님이 강의 초반에 "PPO와 SAC가 현대 RL에서 가장 중요한 두 알고리즘"이라고 위상을 명시했으므로, SAC 강의가 곧 이어질 것으로 보임.
- GRPO 관련 후속: 교수님이 "GRPO를 구현하고 싶다면 더 찾아보라"는 취지로 언급.

