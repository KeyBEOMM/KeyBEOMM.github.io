---
title: "RL summary"
description: "RL"
author: KEYBEOMM
date: 2026-06-15 00:00:00 +0900
categories: [RL]
tags: [RL]
math: true
---

# RL 시험 직전 총망라 정리본 — 전체 모델 한 권

> 범위: 12강(Policy Gradient/Actor-Critic) ~ 24강(Offline RL). **VLA(23강)·World Model(25강) 제외.**
> 목적: 시험 직전에 이 문서 하나만 펼쳐도 "어떤 모델이, 왜 나왔고, 무엇이 핵심이고, 어디서 헷갈리는지"가 잡히게.
> 표기: ★★★ 핵심 / ★★ 중요 / ★ 참고. **"교수님:"** = 강의에서 실제로 강조·반복한 표현. `[보충]` = 강의에 없는 일반지식.

---

## 0. 전체 지도 — 한 장으로 보는 계보

### 0-1. 알고리즘 계보 (교수님이 잡아준 흐름)

```
                          ┌── Value-based ───────────────────────────────┐
                          │   DQN → Double DQN → Dueling DQN → PER        │   (13강)
                          │        (+ Weighted/Averaged/MaxMin DQN)       │
   강화학습(Model-Free) ──┤
                          │   Policy-based / Actor-Critic                 │
                          │   REINFORCE → QAC → A2C(Advantage) → TD-AC     │   (12강)
                          │        → A3C(병렬) / GAE(λ 어드밴티지)         │   (14강)
                          │                                               │
                          │   On-policy 챔피언:  TRPO → PPO → GRPO         │   (16강)
                          │   Off-policy(연속): DPG → DDPG → TD3 → SAC     │   (17·18강)
                          └───────────────────────────────────────────────┘

   모방학습(Imitation) ── BC → DAgger / DART  ┐  (19강)
                          IRL → Apprenticeship → MaxEnt IRL → GAIL  (22강)
                          (LLM 대응: SFT=BC, RLHF=DAgger 정신, RLVR=GRPO)

   오프라인(Offline RL) ── IS 기반(IS·APG·MIS) vs DP 기반(Policy Constraint·Uncertainty·CQL)  (24강)
```

### 0-2. 분류 4축 — 모든 알고리즘은 이 좌표로 정리된다

| 축 | 한쪽 | 반대쪽 | 비고 |
|---|---|---|---|
| **무엇을 학습하나** | Value-based (DQN류) | Policy-based (PG/AC류) | Actor-Critic은 둘 다 |
| **데이터 출처** | On-policy (PPO) | Off-policy (DQN·DDPG·TD3·SAC) | Off-policy = 리플레이 버퍼 |
| **폴리시 형태** | Stochastic (확률분포 출력) | Deterministic (값 하나 출력, DDPG) | SAC는 Stochastic+연속 |
| **액션 공간** | Discrete (객관식) | Continuous (주관식, DDPG/SAC) | 연속이면 DQN 못 씀 |

> **On-policy vs Off-policy ≠ Online vs Offline** (24강에서 교수님이 "내가 먼저 헷갈려서 말한다"고 강조).
> On/Off-policy 둘 다 **온라인**(환경과 상호작용 O). **Offline**은 상호작용 자체가 금지.

### 0-3. 마스터 비교표 (연속 제어 / 정책계열 핵심)

| 알고리즘 | On/Off | 폴리시 | 액션 | 네트워크 | 핵심 한 줄 |
|---|---|---|---|---|---|
| REINFORCE | On | Stochastic | D/C | Policy | return $v_t$ 그대로 가중치 (MC, variance 큼) |
| Q Actor-Critic | On | Stochastic | D | Policy+Q | 가중치를 $Q_w$로 |
| A2C / TD-AC | On | Stochastic | D | Policy+V | 가중치를 Advantage(=TD error δ)로 |
| A3C | On | Stochastic | D | Global+Workers | 병렬 워커로 correlation 해소 |
| TRPO | On | Stochastic | D/C | Policy+V | KL 제약으로 큰 업데이트 차단 (2차 최적화) |
| **PPO** | On | Stochastic | D/C | Policy+V | **clip 하나로** trust region 흉내 (1차) |
| GRPO | On | Stochastic | (LLM) | **Policy만** | critic 제거, 그룹 평균/표준편차로 Advantage |
| DDPG | Off | **Deterministic** | C | **4개** | DPG + NN + DQN(버퍼·타겟) |
| TD3 | Off | Deterministic | C | 6개 | DDPG + Clipped Double-Q + 지연·노이즈 |
| **SAC** | Off | Stochastic | C | Q2+타겟2+Policy | 밸류에 **엔트로피** 더함 → soft |

> 교수님(16강 도입): **"Model-free RL에서 가장 중요한 두 개 = PPO(on-policy)와 SAC(off-policy)."**

---

# Part 1. Value-based — DQN 계열 (13강)

> 프레이밍: "DQN이 너무 성공해서, 그걸 물어뜯으려고(개선하려고) 나온 대표 알고리즘들."
> DQN 복습: Q-Learning 기반 + **Experience Replay**(correlation 해소) + **Target Network**(학습 안정).

## 1-1. Double DQN (DDQN) ★★★

**문제 — Overestimation(과대추정):** DQN은 Q값을 너무 크게 추정. "인플레이션이 심하다."
- 원인: TD 타겟의 **max 연산**. unbiased여도 발생(논문 Corollary 1).
- Jensen 부등식: $\mathbb{E}[\max_a Q] \ge \max_a \mathbb{E}[Q]$ → max 후 기댓값이 항상 크거나 같다.

**해결 — 액션 선택과 값 평가를 분리(decouple):**
$$Y^{DQN}=r+\gamma \max_{a'}Q(s',a';\theta^-)$$
$$Y^{DDQN}=r+\gamma\, Q\!\big(s',\ \underset{a'}{\arg\max}\,Q(s',a';\theta),\ \theta^-\big)$$
- $\theta$(현재망)에서 argmax 액션 **선택**, $\theta^-$(타겟망)에서 그 액션 **값 평가**.

> 교수님(시험 비유): "영어 1등 학생을 뽑아 **그 학생의 수학 점수**를 쓰면, 수학은 다른 사람이 더 높을 수 있다. 그래서 분리하면 항상 max보다 작거나 같다."
> 교수님 평가: "문제 분석은 잘했는데 솔루션은 좀 용두사미."

**함정:** 학습이 진행되면 두 네트워크가 비슷해져 효과↓ → **학습 도중** 차이가 핵심. DDQN도 완벽하진 않음(같은 데이터 학습 → 상관·underestimation 가능).

## 1-2. DDQN 변형들 ★ ("파파라치처럼 꼭 나온다")
| 변형 | 핵심 |
|---|---|
| Weighted Double Q | $Y=\beta Y^{DQN}+(1-\beta)Y^{DDQN}$ (섞기) |
| Averaged DQN | 과거 $k$개 Q의 이동평균 → **variance↓(로우패스 필터)**, 단 메모리 $O(k)$ |
| MaxMin DQN | $N$개 망 중 액션별 **min** 먼저 → 그 중 **max** 선택 ("입찰 최저가") |

## 1-3. Dueling DQN ★★★

**동기 2가지:**
1. DQN은 Q를 **개별** 업데이트 → 한 액션 학습해도 나머지 액션값이 같이 안 큼 → 비효율.
2. **Q보다 State Value(V)가 중요할 때가 있다** (빈 도로 → V에 어텐션, 장애물 앞 → A 활성).

**구조 — Q = V + A 분해:**
$$Q(s,a)=V(s)+A(s,a)\quad(\text{공유 피처} \to \text{V스트림 + A스트림})$$

**Identifiability 문제(학습 안 됨):** V=0,A=(10,12,14)도, V=10,A=(0,2,4)도 같은 Q → 조합이 무한. constraint 없으면 의도대로 안 갈라짐.

**해결 — Mean Subtraction:**
$$Q(s,a)=V(s)+\Big(A(s,a)-\tfrac{1}{|\mathcal A|}\sum_{a'}A(s,a')\Big)$$
- A의 합을 0으로 강제 → V가 진짜 State Value처럼 학습.

> 교수님: "Q=V+A를 그냥 구현하면 학습이 안 되더라." (반드시 mean/max subtraction)

## 1-4. PER (Prioritized Experience Replay) ★★★

**동기(오답노트 비유):** "32개 문제를 무작위로 뽑는 게 최선이냐? 틀린 거·어려웠던 걸 더 봐야 실력이 는다." → **TD error 큰 샘플**을 더 자주.

**핵심 수식:**
$$p_i=|\delta_i|+\epsilon,\qquad P(i)=\frac{p_i^\alpha}{\sum_k p_k^\alpha}$$
- $\alpha=0$ → uniform(기존). $\alpha$↑ → TD error 큰 샘플 집중.
- **Rank-based:** $P(i)\propto 1/\text{rank}(i)$ (값 대신 순위 → 극단값에 둔감).

**IS 보정 (편향 제거):**
$$w_i=\Big(\tfrac{1}{N}\cdot\tfrac{1}{P(i)}\Big)^\beta,\quad \text{정규화}\ w_i/\max_j w_j$$
- $\beta:0\to1$로 annealing(처음엔 우선순위, 나중엔 uniform).

**함정:** Greedy(TD error 0 → 영영 안 뽑힘, ε로 방지). IS 없이 우선순위만 쓰면 bias. α,β 둘 다 하이퍼파라미터.
> [보충] Replay Buffer는 on-policy PPO도 "에피소드 섞기"용으로 쓰기도 함.

---

# Part 2. Policy Gradient → Actor-Critic (12강)

## 2-0. 가장 중요한 한 장 — "가중치로 뭘 쓰느냐" 표 ★★★

$$\nabla_\theta J(\theta)=\mathbb{E}_{\pi_\theta}\big[\nabla_\theta\log\pi_\theta(s,a)\cdot \boxed{?}\big]$$

| 가중치 $?$ | 알고리즘 |
|---|---|
| $v_t$ (실제 return) | **REINFORCE** (MC, variance 큼) |
| $Q_w(s,a)$ | **Q Actor-Critic** |
| $A_w(s,a)=Q-V$ | **Advantage Actor-Critic (A2C)** |
| $\delta$ (TD error) | **TD Actor-Critic** ← 일반적으로 말하는 A2C |
| $\delta e$ (eligibility) | **TD(λ) Actor-Critic** |

> 방향($\nabla\log\pi$)은 고정, **가중치(크기)를 무엇으로 정하냐**가 알고리즘을 가른다.

## 2-1. Policy Gradient 복습 ★★
- $\pi_\theta(s,a)=\mathbb P[a|s,\theta]$. 미분하면 기댓값이 깨짐 → **Likelihood Ratio Trick**으로 복원.
- Score function: $\nabla_\theta\pi_\theta=\pi_\theta\nabla_\theta\log\pi_\theta$ (∵ $\nabla\log x=\nabla x/x$).
- **함정:** REINFORCE의 $v_t$는 **return**(밸류 아님). MC라 에피소드 끝까지 대기 + variance 큼.

## 2-2. Actor-Critic 개요 ★★★
- **Actor**(θ, 폴리시 = 액션) + **Critic**(w, 밸류 = 평가).
> 교수님 비유: **위플래시**(교수=크리틱, 드러머=액터), **SLAM**(위치↔지도처럼 액션↔밸류가 서로 의존, 하나 망가지면 악순환).

## 2-3. Semi-gradient ★★★ (DQN에도 동일 적용)
크리틱 로스 $L=\tfrac12(r+\gamma Q_w(s',a')-Q_w(s,a))^2$에서 **TD 타겟은 상수 취급**, $Q_w(s,a)$만 미분.
$$\nabla_w L=-\delta\,\nabla_w Q_w(s,a)\ \Rightarrow\ w\leftarrow w+\beta\delta\qquad(\textbf{강의자료 표기 — 기준})$$
- [보충] 강의자료는 $\nabla_w L=-\delta$로 축약. 신경망에서 그래디언트 항까지 풀면 $w\leftarrow w+\beta\,\delta\,\nabla_w Q_w(s,a)$ (선형근사 $Q_w=w^\top\phi$면 $\nabla_w Q=\phi$).
> 교수님: "코드에서 **detach**. 벨만 방정식은 TD 타겟을 **따라가는** 거지 바꾸는 게 아니다." (이유: 안정성 / 벨만 의미 / 실용성)

## 2-4. Advantage Function ★★★
$$A(s,a)=Q(s,a)-V(s)$$
- **교수님 직관(평균 대비):** $\sum_a A=0$ — "평균에서 뺀 값이라 합이 0" (모든 액션을 균등 평균으로 볼 때)
- **엄밀한 표기:** $\mathbb{E}_{a\sim\pi}[A^\pi(s,a)]=\mathbb{E}_{a\sim\pi}[Q]-V=0\quad(\because V^\pi(s)=\mathbb{E}_{a\sim\pi}[Q^\pi(s,a)])$
- 단순 합 $\sum_a A=0$은 $V$가 균등평균 $\tfrac1{|\mathcal A|}\sum_a Q$일 때만 성립 → **Dueling DQN의 mean subtraction이 바로 그 균등평균 형태.**
> 시험 점수 비유(평균 대비 ±몇 점). Q=(19800,1000,...)면 다 같이 가지만, A로 바꾸면 **나쁜 방향은 반대로** 간다 → 학습 유리.
> 교수님: "우리가 쓰는 액터 크리틱은 **다 Advantage Actor-Critic**."

## 2-5. A2C → TD Actor-Critic ★★★ (이 강의의 결론)
- A2C 직접 구현은 네트워크 3개(π,Q,V) 필요 → "별로 안 좋다."
- **Baseline 증명:** $\mathbb{E}[\nabla\log\pi\cdot V(s)]=0$ (V는 a에 무관, $\sum_a\nabla\pi=\nabla 1=0$) → V 빼도 bias 없음.
- **핵심: TD error의 기댓값 = Advantage**
$$\mathbb{E}[\delta\mid s,a]=\mathbb{E}[r+\gamma V(s')]-V(s)=Q(s,a)-V(s)=A(s,a)$$
→ **V 네트워크 하나만** 있으면 됨. "지금까지 본 것 중 제일 간단."

## 2-6. TD(λ) Actor-Critic & Eligibility Traces ★★
- n-step Advantage / forward-view λ-가중합 / backward-view eligibility.
- 차이: 방문 시 1을 더하는 게 아니라 $\nabla_\theta\log\pi_\theta(s_t,a_t)$를 더함("그 그래디언트 방향에 연관된 애들만 업데이트").

---

# Part 3. A3C / GAE (14강)

## 3-1. A3C (Asynchronous Advantage Actor-Critic) ★★★
- A2C에 **비동기 병렬** 추가. "새 알고리즘이라기보다 방법론."
- **구조:** Global(Main) Network 하나 + 여러 Worker(스레드). 워커는 각자 다른 환경에서 그래디언트 계산 → Global에 전달 → 업데이트된 가중치 복사받음. **Experience Replay 불필요.**
- **Correlation 해소:** 서로 다른 환경의 워커들이 모아오니 Global 입장에서 들어오는 그래디언트끼리 상관 없음.
> 교수님 비유(행상/수금): "각 워커가 자기 구역에서 수금해서 본점에 납부." / "두 번째 워커는 첫 워커 업데이트도 반영됨." **"이게 A3C의 전부입니다. 전부예요."**
- 구현: 멀티스레드(i7→12스레드), 분산 학습 가능(그래디언트만 주고받음). DQN보다 빠르고 우수.

## 3-2. GAE (Generalized Advantage Estimation) ★★★
- **"GAE = TD(λ)를 Advantage에 적용한 것."** 이미 많은 알고리즘(TD-AC 구현)이 내부적으로 쓰고 있었다.
$$\delta_t=r_t+\gamma V(s_{t+1})-V(s_t)=\hat A_t^{(1)},\qquad \hat A_t^{(n)}=\sum_{l=0}^{n-1}\gamma^l\delta_{t+l}$$
$$\hat A_t^{GAE(\gamma,\lambda)}=(1-\lambda)\sum_{n=1}^{\infty}\lambda^{n-1}\hat A_t^{(n)}=\sum_{l=0}^{\infty}(\gamma\lambda)^l\delta_{t+l}$$
- $\lambda=0$ → $\delta_t$ (bias↑ var↓), $\lambda=1$ → MC (bias↓ var↑). $(1-\lambda)$는 정규화 계수.
- **실제 계산:** 역방향 누적 $\hat A_t=\delta_t+(\gamma\lambda)\hat A_{t+1}$ (맨 뒤부터 앞으로).
> "n-step 늘면 bias↓ variance↑ — TD(λ)와 완전히 같은 트레이드오프."

---

# Part 4. TRPO → PPO → GRPO (16강)

## 4-1. 폴리시 업데이트 크기 문제 ★★★ (강의 전체 도입)
- 너무 작으면 학습 느림, 너무 크면 **Performance collapse**.
> 교수님 비유: **도파민**(A>0 → 더, A<0 → 덜) / **안개 낀 절벽**(한 발 너무 크게 디디면 추락 → "얼마까지 안전하냐"의 답이 TRPO).
> **RL 연쇄 붕괴 = 지도학습과의 결정적 차이:** "RL은 학습한 걸로 데이터를 또 만들어 학습 → 한 번 망가지면 계속 망가진다." (ImageNet은 나쁜 배치 와도 복구됨)

## 4-2. TRPO 복습 ★★
$$J(\theta)=\mathbb{E}_{\pi_{old}}\Big[\tfrac{\pi_\theta(a|s)}{\pi_{old}(a|s)}A_{old}\Big],\quad r_t(\theta)=\tfrac{\pi_\theta}{\pi_{old}},\ r(\theta_{old})=1$$
- **제약형:** maximize 위 식 s.t. $\mathbb{E}[\text{KL}(\pi_{old},\pi_\theta)]\le\delta$. **벌점형:** $-\beta\,\text{KL}$.
- **문제:** 2차 최적화(Fisher matrix, conjugate gradient, line search) → 어렵고, $\beta/\delta$ 하나로 모든 환경 최적 어렵다 → **PPO 동기.**

## 4-3. PPO: Clipped Objective ★★★ (강의 최대 비중)
$$r(\theta)=\tfrac{\pi_\theta}{\pi_{old}},\quad \text{clip}(r,1-\epsilon,1+\epsilon)$$
$$J_{CLIP}=\mathbb{E}\Big[\min\big(r_t\hat A,\ \text{clip}(r_t,1-\epsilon,1+\epsilon)\hat A\big)\Big]$$
- **A>0(좋은 액션):** r 키울수록 J↑, 단 $1+\epsilon$에서 **ceiling**(grad 0).
- **A<0(나쁜 액션):** r을 0쪽으로 줄일수록 J↑("부끄러우니 숨김"), 단 $1-\epsilon$에서 **floor**.
> "이것이 PPO의 전문입니다"를 **세 번** 반복. KL·Fisher·line search 전부 없음, $\epsilon\approx0.2$로 대부분 잘 됨.
- **함정:** A<0일 때 J 높이려면 r을 **줄여야** 함(부호 혼동 주의). clip은 **휴리스틱**(TRPO의 단조개선 보장 없음).

## 4-4. PPO 알고리즘 ★★
```
for k: ① π_k로 trajectory 수집 ② Advantage 추정(value network 필요!) ③ K-step minibatch SGD로 J_CLIP 최대화
```
- 같은 데이터로 **K번** 업데이트(데이터 효율↑). r이 경계 닿으면 grad 0이라 안 변함.
- **PPO = Actor + Critic**(GAE로 Advantage). ← GRPO와의 결정적 차이.
- 5대 미덕: Simplicity / Robustness / Efficiency / Scalability / Generality. 한계: on-policy라 sample-inefficient.

## 4-5. Appendix: PPO-v1 (Adaptive KL) ★
$$L^{KLPEN}=\mathbb{E}[r_t\hat A-\beta\,\text{KL}],\quad d<d_{targ}/1.5\Rightarrow\beta/2,\ \ d>d_{targ}\cdot1.5\Rightarrow\beta\cdot2$$
> 논문 v1=KL, v2=clip. **실험상 KL 안 쓴 clip이 더 좋아서** "PPO=clip 버전."

## 4-6. GRPO (Group Relative Policy Optimization) ★★★
- 배경: LLM RLHF에서 7B×4(policy+value+reward+reference)=28B → **"too expensive."** DeepSeekMath(2024)→DeepSeek-R1(2025).
- **핵심: Critic(value network) 제거.** 같은 입력에서 G개 출력 → 보상들의 평균/표준편차로 Advantage 직접 계산.
$$\hat A_i=\frac{r_i-\text{mean}(r_{1..G})}{\text{std}(r_{1..G})}$$
$$\mathcal J_{GRPO}=\tfrac1G\sum_i\tfrac1{|o_i|}\sum_t\Big\{\min[\rho_{i,t}\hat A,\ \text{clip}(\rho,1-\epsilon,1+\epsilon)\hat A]-\beta D_{KL}[\pi_\theta\|\pi_{ref}]\Big\}$$
- **KL 기준이 PPO와 다름:** PPO는 KL($\pi_{old}\|\pi_{new}$), **GRPO는 KL($\pi_\theta\|\pi_{ref}$)** = **학습 시작 시 초기 모델**과 비교 → catastrophic forgetting 방지.
- **KL estimator (Schulman k3):** $r-\log r-1$ ($r=\pi_{ref}/\pi_\theta$). 항상 ≥0, unbiased.
- **작동 조건 4가지:** ① G개 출력 쉽게 생성 ② 보상이 끝에 한 번(sparse) ③ 출력들 의미 있게 비교 가능 ④ value망이 policy만큼 비싸야 절약 이득.
- GRPO 강점 영역: 수학·코딩·CoT(검증 가능). PPO 적합: 일반 RLHF·글쓰기·dense reward.

## 4-7. PPO vs GRPO ★★
| | PPO | GRPO |
|---|---|---|
| 네트워크 | Policy+Value | **Policy만** |
| Advantage | Critic(GAE) | Group mean/std |
| KL 기준 | $\pi_{old}$ | $\pi_{ref}$(초기) |
| 메모리 | 큼 | 40–60% 절약 |
| 적합 | dense·일반 RLHF | sparse·verifiable |

---

# Part 5. DDPG (+ TD3) (17강)

## 5-1. Deterministic Policy ★★★
- 지금까지 = Stochastic(확률분포). 오늘 = **Deterministic**($a=\mu_\theta(s)$, 값 하나).
> 비유: **객관식(stochastic) vs 주관식(deterministic).** "짜장면" vs "60% 짜장면…".
- **왜 필요?** 6족 로봇(18모터×3선택)=$3^{18}\approx3.87$억 → 연속공간에서 DQN식 이산화는 비현실적. "최적 액션 찾기 자체가 또 다른 최적화 문제."
- Stochastic의 PG 수식($\nabla\log\pi$)은 **확률이 사라져서 못 씀.**

## 5-2. Discounted State Distribution ★★ ("DDPG 쓰는 데 그렇게 중요친 않다")
$$\rho^\mu(s')=\int\sum_{k=1}^\infty \gamma^{k-1}p(s\to s',k,\mu)\rho_0(s)\,ds$$
- Stationary(균등 합) vs **Discounted**($\gamma^{k-1}$ 가중). 쓰는 이유: 수학적으로 떨어짐 / Critic이 이미 discount / deterministic이 $d^\mu$를 깸.
> 비유: **뚜껑 열어 "지금 어디 있니?"** / 경희대 학생 확률(부산+서울+…). "구현 땐 그냥 샘플 뽑으면 끝."

## 5-3. DPG Theorem — Chain Rule ★★★ (핵심 수식)
$$J(\theta)=\mathbb{E}_{s\sim\rho^\mu}[Q^\mu(s,\mu_\theta(s))]$$
$$\nabla_\theta J=\mathbb{E}\big[\underbrace{\nabla_\theta\mu_\theta(s)}_{\theta\to a}\cdot\underbrace{\nabla_a Q^\mu(s,a)}_{a\to Q}\big]\qquad(\textbf{강의자료 표기 — 기준})$$
- [보충] $\nabla_a Q$는 현재 정책이 내는 액션 $a=\mu_\theta(s)$에서 평가하는 것 → 명시하면 $\nabla_\theta J=\mathbb{E}_{s\sim\rho^\mu}\big[\nabla_\theta\mu_\theta(s)\cdot\nabla_a Q^\mu(s,a)\big|_{a=\mu_\theta(s)}\big]$ (강의자료 chain rule 슬라이드엔 이 평가지점이 표시됨).
> 2×2 케이스로 직관: (θ↑→a 변화)×(a↑→Q 변화) 부호 조합으로 θ를 키울지/줄일지 결정.
- **Actor는 Gradient Ascent**(J 최대), **Critic은 Descent**(TD error 최소). 부호 주의.

## 5-4. DDPG = DPG + NN + DQN ★★
- DPG(Silver 2014, 이론) → DDPG(Lillicrap 2015, NN 구현). Model-free / Off-policy / Actor-Critic / **연속 액션**.
- "밸류 네트워크는 그냥 DQN(TD error 학습)."

## 5-5. Exploration & OU Noise ★★
- Deterministic은 같은 state→같은 action→**탐색 0**. 해결: $a=\mu_\theta(s)+\mathcal N$ (원논문 **OU Noise**). 노이즈는 환경 상호작용용, 학습 Q계산엔 미반영.

## 5-6. 4-네트워크 구조 & Soft Update ★★★ ("가장 헷갈리는 부분")
| 파라미터 | 모델 | 역할 |
|---|---|---|
| $\theta^\mu$ | Actor | s→a |
| $\theta^{\mu'}$ | **Target Actor** | TD타겟의 next action |
| $\theta^Q$ | Critic | (s,a)→Q |
| $\theta^{Q'}$ | **Target Critic** | TD타겟의 next Q |

> **왜 Actor에도 타겟이?** "타겟 크리틱에 들어갈 next action도 **과거 액터**에서 뽑는 게 낫다. **타겟끼리 겸상.**"

**Soft Update:** $\theta'\leftarrow\tau\theta+(1-\tau)\theta'$ ($\tau\ll1$, 예 0.001). DQN의 hard update와 달리 매 스텝 조금씩.

**짝 맞추기 (시험 포인트):**
| | next action | Q값 |
|---|---|---|
| TD Target 계산 | Target Actor μ′ | Target Critic Q′ |
| Actor 업데이트 | Behavior μ | **Behavior Critic Q** |

**Take-home:** ① deterministic 업데이트=DPG/Chain Rule ② exploration=OU Noise ③ 네트워크 **4개**.

## 5-7. TD3 (Twin Delayed DDPG) ★★ — 기말 객관식 단골 (Target Policy Smoothing)
> [보충] 전용 강의 정리본 없음 → 일반지식 기반(Fujimoto et al. 2018). DDPG의 **과대추정·불안정**을 3가지 트릭으로 개선. SAC가 ①을 차용.

**① Clipped Double-Q (Twin Critics):** Q망 2개 → TD 타겟에 **둘 중 min** 사용 → 과대추정 억제.
$$y=r+\gamma\min_{i=1,2}Q_{\theta_i'}(s',\tilde a)\qquad[\text{plain}]\ \ y = r + γ·min(Q1', Q2')$$

**② Target Policy Smoothing:** **타겟 액션**에 clip된 노이즈를 더해 Q의 잘못된 뾰족한 peak를 과신하지 않게 함(정규화). 직관: "비슷한 액션은 비슷한 Q값."
$$\tilde a=\mu_{\theta'}(s')+\epsilon,\quad \epsilon\sim\text{clip}(\mathcal N(0,\sigma),-c,+c)\qquad[\text{plain}]\ \ ã = μ'(s') + clip(N(0,σ), -c, +c)$$

**③ Delayed Policy Update:** Critic을 d번(보통 2번) 업데이트할 때마다 **Actor·타겟망은 1번만** → Critic이 안정된 뒤 Actor 갱신.

**객관식 "옳지 않은 설명" 단골 함정 (전부 ✗ = 틀린 보기):**
- 노이즈를 behavior(실제 환경) 액션에 더한다 → ✗ (그건 exploration noise. smoothing은 **타겟 액션**에)
- 타겟 노이즈를 clip 하지 않는다 → ✗ ($-c\sim+c$로 clip)
- Actor를 Critic보다 **자주** 업데이트 → ✗ (Actor가 **덜** 자주 = delayed)
- 타겟에 min 대신 **max** → ✗ (min)
- DDPG와 exploration 방식이 동일하다 → ✗ (DDPG=액션에 OU noise / TD3 smoothing=**타겟**에 가우시안+clip)

---

# Part 6. SAC (Soft Actor-Critic) (18강)

> 프레이밍: **"모든 것이 다 엔트로피 탓."** PPO(안정)와 DDPG/TD3(샘플효율)의 장점만 → 답은 "엔트로피를 밸류에 더하기" 하나에서 파생.

## 6-1. 배경: PPO vs DDPG/TD3 딜레마 ★★
- PPO: stochastic/on-policy/안정/비효율. DDPG: deterministic/off-policy/효율/**불안정(아예 학습 실패 잦음)**.
- Off-policy 불안정 3원인: 분포 불일치 / Q 과추정 / **policy collapse**(좁은 액션에 갇힘).

## 6-2. Soft Value Function ★★★ ("아이디어의 거의 전부")
$$\boxed{V(s_t)=\mathbb{E}_{a\sim\pi}[Q(s_t,a)]+\alpha H(\pi(\cdot|s_t))=\mathbb{E}_{a\sim\pi}[Q(s_t,a)-\alpha\log\pi(a|s_t)]}$$
$$Q(s_t,a_t)=r_t+\gamma\mathbb{E}_{s_{t+1}}[V(s_{t+1})]$$
> "Q 기댓값에 **엔트로피까지 더한 게** 밸류. 확률이 고르게 퍼질수록 밸류↑."

## 6-3. "Soft"의 의미 = Softmax ★★★ (교수님 직접 유도)
$\max_\pi \sum\pi Q+\alpha H$ s.t. $\sum\pi=1$를 라그랑지안+정류조건으로 풀면:
$$\pi^*(a|s)=\frac{\exp(Q(s,a)/\alpha)}{\sum_{a'}\exp(Q(s,a')/\alpha)}=\text{softmax}(Q/\alpha)$$
> "밸류를 최대로 만드는 폴리시가 **Softmax** 분포 → 그래서 **Soft** Actor-Critic." (분모=Partition Function)

## 6-4. Soft Actor — KL 최소화 ★★★
현재 π를 목표 $\pi^*\propto\exp(Q/\alpha)$에 맞추도록 KL 최소화 → ($Z$는 φ무관해 제거):
$$\boxed{L_\pi(\phi)=\mathbb{E}_{s\sim D,a\sim\pi_\phi}[\alpha\log\pi_\phi(a|s)-Q_\theta(s,a)]}$$
> **함정:** "엔트로피를 액션·밸류에 따로 넣는 게 아니다. **밸류에 엔트로피를 넣으니 최적 액션이 Softmax**가 되고, 그걸 달성하려다 이 식이 나온 것."

## 6-5. Soft Critic ★★ (TD3 Clipped Double-Q 차용)
$$y_t=r+\gamma\mathbb{E}_{a'}\big[\min_{i=1,2}Q_{\bar\theta_i}(s',a')-\alpha\log\pi(a'|s')\big]$$
$$J_Q=\mathbb{E}\big[(Q_{\theta_1}-y)^2+(Q_{\theta_2}-y)^2\big],\qquad \bar\theta\leftarrow\tau\theta+(1-\tau)\bar\theta$$

## 6-6. Reparameterization Trick ★★
- 문제: 랜덤 샘플링된 액션은 미분 불가. 해결:
$$a_t=f_\phi(\varepsilon;s)=\mu_\phi(s)+\sigma_\phi(s)\cdot\varepsilon_t,\quad\varepsilon_t\sim\mathcal N(0,I)$$
> "모든 게 식이다. 랜덤이라는 의미 자체는 미분 안 됨." VAE에서 쓰던 그 트릭. → 이제 φ로 역전파 가능.

## 6-7. 정리
- 네트워크: $Q_{\theta_1},Q_{\theta_2}$ + 타겟 2개 + Policy $\pi_\phi$(Gaussian $\mu,\sigma$) + 온도 α 자동조정. **연속 액션** 대상(이산 SAC는 2019, 샘플링 없이 모든 액션 Q 출력).
> 위상: "온폴리시는 **PPO**, 오프폴리시는 **SAC**를 가장 많이 쓴다."

---

# Part 7. Imitation Learning-1: BC / DAgger / DART (19강)

> "한 학기 중 제일 쉬운 강의." IL = 리워드 최대화가 아니라 **전문가 행동 모방**. 종류: BC / IRL / GAIL.

## 7-1. Behavioral Cloning (BC) ★★★
- **"결국 Supervised Learning."** state=입력, expert action=정답. 데모 $D=\{(s_i,a_i)\}$.
$$\arg\min_\theta\mathbb{E}_{(s,a^*)\sim P^*}L(a^*,\pi_\theta(s))\quad(\text{1-step deviation error})$$
> 비유: 운전 게임 기록으로 학습 / 폴리시=ChatGPT. "리워드가 없어지는 게 아니라 **정답과의 차이**가 로스."

## 7-2. 핵심 문제: Distribution Shift / Covariate Shift ★★★ (가장 많은 시간)
- BC는 IID 가정 위반: 시퀀셜(독립 X) + 전문가 분포≠롤아웃 분포(동일분포 X).
$$P_{train}(s)\ne P_{test}(s),\quad P(a|s)\ \text{불변}$$
> "전문가는 항상 가운데만 봐서, **벗어난 상태는 배울 기회가 없다.** 한 번 멀어지면 복구 못 함." 해결: IS / data aug / domain adaptation → DAgger·DART.

## 7-3. ALVINN (1988) ★★ — BC 최초 성공(CMU, NeurIPS 1988). 이미지+라이다 → 45방향 출력, data aug(rotation/shift).

## 7-4. DAgger (Dataset Aggregation) ★★★ "네 줄이 다"
```
1. 사람 데이터 D로 π 학습
2. π 롤아웃 → 방문 상태 D_π 수집
3. 그 상태들에 전문가가 라벨(action) 부여
4. D ← D ∪ D_π  (반복)
```
- 핵심: **내가 실제로 방문할 상태**에 라벨 → train/test 분포 일치.
> "전문가 10시간을 한 번에(BC) vs 1시간씩 쪼개 라벨 반복(DAgger)." **Bottleneck:** 5분×30FPS=9000장 라벨링 → 비현실적.
- 실용 변형: Automated Expert(룰), SafeDAgger(불확실할 때만 쿼리), downsampling, HG-DAgger(위험할 때만 개입).

## 7-5. DART ★★ — **데이터 수집 시 노이즈 주입**해 분포를 넓힘. "전문가가 가라는 대로 안 가고 노이즈 섞어 → 벗어난 상황의 복구 액션도 자동 수집." (off-policy식, 안전·피드백 부담↓)

## 7-6. BC Summary ★★
- 장점: 간단·효율. 단점: distribution mismatch, 장기 계획 X.
- 권장: 1-step deviation이 치명적이지 않고 데모가 state space 커버할 때. 비권장: 누적 오차가 catastrophic일 때.

## 7-7. LLM 연결 (SFT/RLHF) ★★
- **SFT = 텍스트 BC**(프롬프트→정답 토큰, cross-entropy).
- **SFT vs RLHF:** SFT는 답을 **정해줌**, RLHF는 모델이 낸 답을 **평가(랭킹)**. RLHF = **DAgger 정신**(내 출력에 라벨).
- RLHF 파이프라인: ①SFT → ②Reward Model(인간 랭킹) → ③RL(PPO). **RLVR**(수학·코딩 자동 검증, GRPO).
- 스펙트럼: `BC/SFT → DAgger → IRL/GAIL → RLHF` (action-level → preference-level).

---

# Part 8. Imitation Learning-2: Inverse RL & GAIL (22강)

> ※ 전사본 없이 슬라이드만 제공된 회차 → 중요도는 슬라이드 강조 신호 기준.
> IRL = BC처럼 행동을 베끼는 게 아니라 **행동 뒤의 보상 함수(의도)를 역으로 복원.**

## 8-1. IRL이란 — RL을 거꾸로 ★★★
| | 입력 | 출력 |
|---|---|---|
| RL | 보상 함수 | 최적 정책 |
| **IRL** | (전문가) 최적 정책 | **보상 함수** |
- 2단 구조: **Reward 복원 → 그 보상으로 RL → Policy.** (dynamics/MDP를 안다고 가정.)

## 8-2. "전문가 행동에 그냥 높은 보상 주면 안 되나?" ★★★ (동기, 강조 최다)
안 되는 3이유: ① BC와 동일(일반화 X) ② 보상은 **장기 return** 정의(전문가는 지금 손해→나중 이득) ③ **모호성**(전문가를 최적으로 만드는 보상 무한, 예 $r\equiv0$).
IRL이 하는 일: ① 보상을 **feature 함수로 매개변수화**($r_w(s)$) ② **원리로 모호성 해소**(max-margin/MaxEnt/adversarial) ③ 복원 보상으로 RL → **의도 복원.**

## 8-3. Feature Based Reward (선형 가정) ★★★
가정 3개: 전문가 최적 / dynamics 앎 / **보상 선형** $R(s)=w^T\phi(s)$.
$$\mathbb{E}\big[\textstyle\sum\gamma^t R(s_t)|\pi\big]=w^T\underbrace{\mathbb{E}[\sum\gamma^t\phi(s_t)|\pi]}_{\mu(\pi)\,=\,\text{feature expectation}}$$
$$\boxed{\text{Find }w^*\ \text{s.t. }w^{*T}\mu(\pi^*)\ge w^{*T}\mu(\pi)\ \ \forall\pi}$$

## 8-4. Feature Matching (Abbeel & Ng 2004) ★★★ (이론 중핵)
$$\|\mu(\pi)-\mu(\pi^*)\|_1\le\epsilon\ \Rightarrow\ |w^T\mu(\pi)-w^T\mu(\pi^*)|\le\epsilon\ (\forall\|w\|_\infty\le1)$$
> **진짜 보상을 몰라도, feature expectation만 맞추면 전문가만큼 잘한다.** → GAIL의 occupancy matching으로 이어짐.

## 8-5. Apprenticeship Learning (max-margin) ★★
- IRL=보상까지 / **Apprenticeship=정책까지**. 반복: max-margin으로 $w$ 찾기 → 그 보상으로 RL → feature expectation 추가 → 반복.
$$t^{(i)}=\max_{\|w\|\le1}\min_j w^T(\mu_E-\mu^{(j)}),\quad t^{(i)}\le\epsilon\Rightarrow\text{종료}$$
- **함정:** 매 iteration마다 RL을 통째로 풀어야 함 → 비쌈 → GAIL 동기.

## 8-6. Maximum Entropy IRL ★★ — 보상 vs 정책의 **zero-sum 게임** + 엔트로피 항으로 모호성 해소. 구조가 **GAN의 min-max와 동일** → GAIL로 연결.

## 8-7. GAIL (Ho & Ermon 2016) ★★★
- **보상 복원을 건너뛰고 정책 직접 학습.** IRL+RL ≡ **occupancy measure 매칭**($\rho_\pi$를 $\rho_E$와 구별 불가하게).
| GAN | GAIL |
|---|---|
| Real data | Expert demos $\rho_E$ |
| Generator $G$ | Policy $\pi_\theta$ ($\rho_\pi$) |
| Discriminator $D$ | $D_w(s,a)$ (전문가 vs 정책 구별) |
| — | **surrogate reward** $r=-\log D_w(s,a)$ → TRPO/PPO로 정책 갱신 |

## 8-8. IL 세 갈래 비교표 ★★
| | Direct Policy 학습 | Reward 학습 | 환경 접근 | Interactive 전문가 | 사전수집 데모 |
|---|---|---|---|---|---|
| BC | Yes | No | No | No | Yes |
| Interactive IL(DAgger) | Yes | No | Yes | Yes | Optional |
| **IRL** | **No** | **Yes** | Yes | No | Yes |
- IRL만 reward 학습. IRL은 정책 직접 학습 X(보상 거침), 환경 접근 필요(RL 돌려야 함).

---

# Part 9. Offline RL (24강)

> "두 학기 중 한 학기를 할 만큼 큰 분야를 한 시간에 압축." 학습 목표 3개: **무엇/온라인과 차이/이슈**.

## 9-1. 정의 ★★★
- **현재 학습한 폴리시를 환경에서 단 한 번도 실행 못 함.** 정적 데이터셋 $D$(behavior policy $\pi_\beta$ 수집)만으로 학습. **목적(objective)은 온라인과 동일**, 제약만 다름.
> "선물로 받은 만 개 트랜짓으로 폴리시를 만들었는데 좋은지 알 수가 없다." / 모의고사만 보고 수능 못 보는 격.

## 9-2. vs Off-Policy ★★★ (가장 헷갈림)
- 공통: 둘 다 $D$에서 학습. 차이: Off-Policy는 새 데이터를 **버퍼에 계속 추가** / Offline은 추가 금지.
- **핵심 = 피드백 루프(self-correction):** Off-Policy는 나쁜 행동 결과가 버퍼에 들어가 "이러면 안 돼"라고 과대추정을 교정. **Offline은 이 장치가 없다.** 탐험도 불가(데이터에 없는 action3의 값은 영영 모름).

## 9-3. vs Behavior Cloning ★★★
- 공통: 사전 데이터·상호작용 X. 차이: **BC=분포 카피**(데이터 optimal 가정), **Offline RL=return 최대화**.
> 교수님 직접 예시: $(s_1,a_1)$ 10개(r=1), $(s_1,a_2)$ 1개(r=10) → **BC는 90%로 $a_1$ 따라함**, **Offline RL은 $a_2$**(return 큼).

## 9-4. 왜 어려운가 = 한마디로 OOD ★★★
4가지: ① Lack of Exploration ② Distribution Shift ③ **Extrapolation Error**(데이터 없는 샘플에 과대 value, 교정 불가) ④ Counterfactual Reasoning.
- Extrapolation 두 원인: (a) Bellman 타겟 **max** 연산이 없는 action의 과대 Q를 따라감 (b) **네트워크 일반화**(테이블과 달리 안 본 곳까지 흔들림).
> **"한마디로 그냥 OOD입니다. OOD. OOD 문제예요."** (SAC가 잘 되는 건 엔트로피→탐험인데, 오프라인은 그걸 못 함.)

## 9-5. 접근법 분류 ★★★
- **① IS 기반:** $\pi_\beta$ 데이터로 IS reweight. $\pi_\beta$는 BC(카운팅)로 추정, **high variance**.
$$J_\pi=\mathbb{E}_{\tau\sim\pi_\beta}\Big[\prod_t\tfrac{\pi(a_t|s_t)}{\pi_\beta(a_t|s_t)}R(\tau)\Big]$$
- **② DP 기반:** value 학습 + bootstrapping. 도전과제 = **Extrapolation error**. → **"실제론 DP를 많이 쓰는 추세."**

## 9-6. IS 기반 3종 & 트레이드오프 ★★★
| 방법 | Variance | Bias | 핵심 |
|---|---|---|---|
| **IS** (per-decision: 미래 weight 버림, $\prod_0^t$) | 큼 | 작음 | ratio 정확히 알면 이상적 |
| **APG** | **가장 작음** | **가장 큼** | product ratio "**안 합니다. 버립니다.**" state만 $\pi_\beta$, action은 내 폴리시. $J=\mathbb{E}_{s\sim d^{\pi_\beta},a\sim\pi}[Q^\pi]$ |
| **MIS** | 중간 | 중간 | state-level ratio $\rho(s)=\tfrac{d^\pi(s)}{d^{\pi_\beta}(s)}$만 보정(학습으로 구함) |

## 9-7. DP 재정의 + 해결책 3종 ★★★
> **용어 주의:** 3월의 DP(planning)와 다름. 여기선 **Bellman으로 value를 한 스텝씩 업데이트하는 부류**(DQN 등)를 통칭. (Sergey Levine 계열)

세 해결책 공통 동기 = **OOD를 너무 후하게(높게) 보지 않게 엄격하게.**
1. **Policy Constraint** ★★★: $\pi$를 $\pi_\beta$에 가깝게 → OOD action을 Q에 query 안 하게. $\pi_{k+1}=\arg\max\mathbb{E}[Q]$ **s.t. $D(\pi,\pi_\beta)\le\epsilon$** (KL/TV).
2. **Uncertainty-based** ★★: 앙상블 → value에서 **variance 빼기**. (10,10,10)=신뢰↑ 유지 / (10,0,−10)=variance↑ 깎임.
3. **Conservative (CQL)** ★★: 데이터에 없는 action의 Q를 **보수적으로 낮춤**.
$$\min_Q\ \alpha\big(\mathbb{E}_{s\sim D,a\sim\pi}[Q]-\mathbb{E}_{s,a\sim D}[Q]\big)+\text{TD Error}$$
(폴리시가 뽑는 action의 Q↓, 데이터에 실제 있는 action의 Q는 살림.)

---

# 시험 직전 압축 — 교수님이 "강조/헷갈린다"고 한 것 모음

**한 줄 등식·정의 (반드시):**
- TD error 기댓값 = Advantage: $\mathbb{E}[\delta|s,a]=Q-V=A$ → **V망 하나로 A2C** (12강 결론)
- Semi-gradient = TD 타겟 **detach**(상수 취급) (12·전반)
- GAE = TD(λ)를 Advantage에 적용, $\hat A^{GAE}=\sum(\gamma\lambda)^l\delta_{t+l}$ (14강)
- DDQN: argmax는 현재망, 값은 타겟망 → max 과대추정 차단 (13강)
- Dueling: Q=V+A는 **mean subtraction 필수**(identifiability) (13강)
- PER: $p=|\delta|+\epsilon$, $P\propto p^\alpha$, IS $w$로 bias 보정 (13강)
- PPO clip: A>0 ceiling($1+\epsilon$), A<0 floor($1-\epsilon$) — **부호 헷갈림 주의** (16강)
- GRPO: critic 제거, $\hat A=(r-\text{mean})/\text{std}$, KL은 $\pi_{ref}$(초기) 기준 (16강)
- DDPG: 네트워크 **4개**, 타겟끼리 겸상, Actor=ascent/Critic=descent, OU Noise (17강)
- SAC: $V=\mathbb{E}[Q]+\alpha H$ → 최적정책=Softmax(Q/α) → Actor loss=$\mathbb{E}[\alpha\log\pi-Q]$ (18강)
- BC 핵심 문제 = Covariate Shift($P_{train}(s)\ne P_{test}(s)$) (19강)
- DAgger = 내가 방문할 상태에 라벨 / DART = 수집 시 노이즈 (19강)
- IRL: feature expectation 매칭이면 보상 몰라도 충분 (Abbeel&Ng) (22강)
- GAIL: occupancy 매칭, surrogate reward $-\log D$, GAN 구조 (22강)
- Offline RL 어려움 = **OOD** / IS는 variance / DP는 extrapolation error (24강)

**교수님이 직접 "헷갈린다"고 경고한 지점:**
- Q-AC(Q망) vs TD-AC(V망) 혼동 금지 (12강)
- A2C 이름이 붙은 구현 대부분이 실제론 **TD Actor-Critic** (12강)
- PPO A<0 케이스(r을 줄여야 J↑) (16강)
- SFT(답 정해줌) vs RLHF(답 평가) (19강)
- **Off-Policy ≠ Offline** / **Exploration ≠ Extrapolation** (24강, 교수님 본인도 정정)

**"중요하지 않다"고 명시한 것 (시험 비중↓):**
- Compatible Function Approximation (12강, "Not Important")
- Discounted State Distribution의 직접 구현 (17강)
- "SAC가 진짜 AC냐" 논쟁 (18강)
- Offline의 PG-with-IS 수식 변형 (24강, "읽어보면 된다")

---

## 부록 — 핵심 수식 한 페이지 (커닝 페이퍼)

```
PG 정리:   ∇J = E[∇logπ · ?]   ? = v_t / Q_w / A_w / δ / δe
Score:     ∇π = π·∇logπ
A2C 핵심:  E[δ|s,a] = Q-V = A,   δ = r+γV(s')-V(s)
Semi-grad: w ← w + βδ∇_w Q   (타겟 detach)
GAE:       Â = Σ(γλ)^l δ_{t+l},  Â_t = δ_t + γλ·Â_{t+1}
DDQN:      Y = r + γ Q(s', argmax_a Q(s',a;θ); θ⁻)
Dueling:   Q = V + (A - mean_a A)
PER:       p=|δ|+ε,  P(i)=p^α/Σp^α,  w=(1/(N·P))^β
TRPO:      max E[r·Â]  s.t.  E[KL(π_old,π)] ≤ δ
PPO:       J = E[min(r·Â, clip(r,1-ε,1+ε)·Â)]
GRPO:      Â=(r-mean)/std,  KL(π_θ‖π_ref),  k3: r-logr-1
DPG:       ∇J = E[∇_θ μ(s) · ∇_a Q(s,a)]
DDPG soft: θ' ← τθ + (1-τ)θ'
SAC V:     V = E_a[Q - α logπ]
SAC π*:    π* = softmax(Q/α)
SAC actor: L = E[α logπ_φ(a|s) - Q(s,a)]
SAC repar: a = μ_φ(s) + σ_φ(s)·ε,  ε~N(0,I)
BC:        argmin E[L(a*, π_θ(s))]
Covariate: P_train(s) ≠ P_test(s),  P(a|s) 불변
IRL:       find w*:  w*ᵀμ(π*) ≥ w*ᵀμ(π) ∀π
Feature match: ‖μ(π)-μ(π*)‖₁ ≤ ε
GAIL:      r(s,a) = -log D_w(s,a)
Offline IS: J = E_{π_β}[ Π (π/π_β) · R ]
Offline APG:J = E_{s~d^πβ, a~π}[ Q^π(s,a) ]   (ratio 버림)
Offline MIS:ρ(s)=d^π(s)/d^πβ(s)
CQL:       min_Q α(E_{a~π}[Q] - E_{a~D}[Q]) + TD Error
```
```
액션 공간:  Discrete → DQN류 / Stochastic-PG     Continuous → DDPG·TD3·SAC
On-policy:  REINFORCE, A2C/A3C, TRPO, PPO, GRPO   (버퍼 X, 쓰고 버림)
Off-policy: DQN류, DDPG, TD3, SAC                 (리플레이 버퍼)
온라인:     On/Off-policy 둘 다                    오프라인: 상호작용 금지
```
```
타겟 네트워크 쓰는 곳:  DQN(hard) / DDPG·TD3·SAC(soft τ)
엔트로피·소프트:        SAC(밸류에 +αH), MaxEnt IRL
Clipped Double-Q(min):  TD3 → SAC가 차용
GAN 구조(min-max):      MaxEnt IRL, GAIL
critic 없음:            REINFORCE, GRPO
```

---

# 기출 기반 예상문제 + 모범답안 (올해 범위 보정)

> **올해 기말 범위 = Actor-Critic부터.** (중간고사에서 PG 유도 + DQN을 이미 봄)
> 작년처럼 **PG가 중간에 나왔으므로 기말엔 유도/증명 문제 없음**(작년 평가 확인).
> 교수님이 문제를 거의 안 바꿈 → 아래는 재작년·작년 기출을 올해 범위로 필터링한 예측.
>
> - **제외 가능성↑(중간에 봄):** PG likelihood-ratio 유도 / DQN 기법·off-policy 이유·pseudocode / NFQ→DQN
> - **그대로 사정권:** AC(목적함수·네트워크 수), DDQN, Dueling, PER, A3C/GAE, TRPO, PPO, GRPO, DDPG, **TD3**, SAC, IL(BC↔IRL 개념)
> - **불확실:** Offline RL(작년 미출제, 올해 강의는 됨 → 개념만 대비)

---

### Q1 [매칭·서술] 각 목적함수의 가중치(method)와 필요한 네트워크 수를 적으시오. (REINFORCE / Q Actor-Critic / A2C / TD Actor-Critic) — **★★★ 거의 확정**

| 알고리즘 | ∇J의 가중치 | 네트워크 수 |
|---|---|---|
| REINFORCE | return v_t (MC) | **1** (Policy) |
| Q Actor-Critic | Q_w(s,a) | **2** (Policy + Q) |
| A2C (직접 구현) | A = Q − V | **3** (Policy + Q + V) |
| TD Actor-Critic | δ = r + γV(s') − V(s) | **2** (Policy + V) |

> 핵심 한 줄: `E[δ|s,a] = Q − V = A` 이므로 V망 하나로 Advantage를 얻음 → 보통 "A2C"라 부르는 구현이 실제론 **TD Actor-Critic**(네트워크 2개).

### Q2 [서술] Semi-gradient란 무엇이며 왜 쓰는가?
- L = ½δ², δ = r + γQ_w(s',a') − Q_w(s,a) 에서 **TD 타겟(r+γQ_w(s',a'))을 상수로 취급(detach)** 하고 Q_w(s,a)만 미분. 결과: `w ← w + βδ∇_w Q_w(s,a)` (강의자료 표기 `w ← w + βδ`).
- 이유 3가지: ① 타겟이 계속 움직이면 학습 불안정 ② 벨만 방정식은 타겟을 **따라가는** 것이지 바꾸는 게 아님 ③ 실용적으로 더 빠르고 잘 됨.

### Q3 [수식] DDQN의 TD target을 작성하시오. — **★★★ 매년 출제**
- `Y^DDQN = r + γ · Q(s', argmax_a' Q(s',a';θ); θ⁻)`
- 액션 **선택**은 현재망 θ, 그 액션의 **평가**는 타겟망 θ⁻ → max 과대추정 차단. (DQN: `Y = r + γ max_a' Q(s',a';θ⁻)`)

### Q4 [서술] PER에서 α 값에 따라 어떤 방식과 비슷해지는가? — **★★★ 기출 그대로**
- P(i) = p_i^α / Σ p_k^α, p_i = |δ_i| + ε.
- **α = 0 → 모든 확률 동일 = uniform sampling = 기존 Experience Replay.**
- **α ↑ (→ 큰 값) → TD error 큰 샘플에 집중 = greedy(우선순위) 샘플링.**
- (β는 IS 보정: β=0 보정 없음 → β=1 완전 보정(uniform 효과), 학습 중 0→1 annealing)

### Q5 [수식] Trust Region(TRPO)의 목적함수와 제약을 작성하시오. — **★★★ 기출 그대로**
- maximize_θ  E[ (π_θ(a|s) / π_old(a|s)) · Â ]
- subject to  E[ KL(π_old(·|s) ‖ π_θ(·|s)) ] ≤ δ
- (벌점형: 목적함수에 −β·KL 추가)

### Q6 [계산] PPO 목적함수 J_CLIP의 최대값을 구하시오. (ε, Â, π_old 주어짐) — **★★★ 계산형 단골**
- r = π_new / π_old,  J = min( r·Â , clip(r, 1−ε, 1+ε)·Â )
- **Â > 0:** r을 키울수록 J↑, 단 r = 1+ε에서 **천장(ceiling)** → **J_max = (1+ε)·Â**. (이후 grad=0)
- **Â < 0:** r을 줄일수록 J↑, 단 r = 1−ε에서 **바닥(floor)** → 더 못 내려감, 한계값 **(1−ε)·Â**.
- 예: π_old=0.5, Â=+2, ε=0.2 → r=1.2(π_new=0.6)에서 clip, **J_max = 1.2 × 2 = 2.4**. π_new=0.7(r=1.4)이어도 J=2.4로 동일(clipped).

### Q7 [코드 매칭·서술] DDPG에서 ⓐ TD target 계산과 ⓑ Actor 업데이트가 각각 어느 네트워크를 쓰는가? — **★★ 기출(코드 라인 고르기)**
- ⓐ **TD target:** `y = r + γ Q'(s', μ'(s'))` → next action = **Target Actor μ'**, next Q = **Target Critic Q'**. (타겟끼리 겸상)
- ⓑ **Actor 업데이트(ascent):** `θ^μ ← θ^μ + β ∇_θ μ(s) · ∇_a Q(s,a)|_{a=μ(s)}` → **Behavior Actor μ + Behavior Critic Q**.
- Critic은 descent(TD error 최소), Actor는 ascent(목적함수 최대). Soft update: `θ' ← τθ + (1−τ)θ'`.

### Q8 [객관식] SAC 설명 중 옳지 않은 것을 모두 고르시오. — **★★ 기출 유형**
틀린 보기(✗) 후보 / 맞는 보기(✓):
- ✗ "SAC는 deterministic 정책을 출력한다" → stochastic(가우시안 μ,σ)에서 샘플링.
- ✗ "엔트로피를 밸류와 정책에 인위적으로 따로 더한다" → **밸류 정의에 엔트로피를 넣으니 최적정책이 softmax(Q/α)가 되는 것**(파생).
- ✗ "on-policy 알고리즘이다" → off-policy(리플레이 버퍼).
- ✓ "Reparameterization trick으로 정책을 미분 가능하게 한다" (a = μ + σ·ε).
- ✓ "Clipped Double-Q를 TD3에서 차용했다."
- 핵심식: V = E[Q − α·logπ], π* = softmax(Q/α), Actor loss = E[α·logπ − Q].

### Q9 [객관식] TD3의 Target Policy Smoothing 설명 중 옳지 않은 것을 모두 고르시오. — **★★ 기출 그대로**
- ✗ 노이즈를 실제 환경에서 하는 behavior 액션에 더한다 (→ 타겟 액션에 더함)
- ✗ 타겟 노이즈를 clip 하지 않는다 (→ −c~+c clip)
- ✗ Actor를 Critic보다 자주 업데이트한다 (→ Actor가 덜 자주, delayed)
- ✗ 타겟에 두 Q 중 max를 쓴다 (→ min)
- ✓ 노이즈는 `ã = μ'(s') + clip(N(0,σ), −c, +c)`로 타겟 계산에만 들어간다.

### Q10 [서술] Behavior Cloning과 Inverse RL objective의 차이를 서술하시오. — **★★★ 기출 그대로**
- **BC:** `argmin_θ E_{(s,a*)~P*}[ L(a*, π_θ(s)) ]` — 전문가 **행동을 직접 복제**(supervised, 1-step). 보상 함수 안 배움. 단점 = covariate shift(P_train(s) ≠ P_test(s)).
- **IRL:** 전문가를 최적으로 만드는 **보상 R을 역으로 복원** → `find w*: w*ᵀμ(π*) ≥ w*ᵀμ(π) ∀π`. 행동이 아니라 **의도(보상)** 를 배우고, 복원한 R로 RL을 돌려 정책 획득(2단 구조). 일반화·장기성·보상 모호성 해결을 지향.

### Q11 [서술] (A3C/GAE — 범위 내, 기출엔 없었으나 대비) GAE와 TD(λ)의 관계, λ=0/1 극한은?
- GAE = **TD(λ)를 Advantage에 적용.** `Â^GAE = Σ_l (γλ)^l · δ_{t+l}`, 재귀형 `Â_t = δ_t + γλ·Â_{t+1}`.
- λ=0 → δ_t (bias↑, variance↓) / λ=1 → MC (bias↓, variance↑).
- A3C: Global망 + 여러 Worker 비동기 병렬 → 워커들이 서로 다른 환경에서 모아 **correlation 해소**(Experience Replay 불필요).

### Q12 [객관식·서술] (GRPO — 범위 내, 최신) PPO와 GRPO의 차이는?
- GRPO는 **Critic(value망) 제거**, 같은 입력에서 G개 출력의 보상으로 `Â = (r − mean) / std`.
- KL 기준: PPO는 π_old(직전), **GRPO는 π_ref(학습 초기 모델)** → catastrophic forgetting 방지.
- 강점: 메모리 40–60% 절약, sparse·verifiable(수학·코딩) 도메인. 한계: dense reward는 PPO가 유리.
