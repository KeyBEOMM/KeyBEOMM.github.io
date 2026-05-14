---
title: "12강 Actor-Critic"
description: "Actor-Critic"
author: KEYBEOMM
date: 2026-05-14 20:00:00 +0900
categories: [RL]
tags: [RL, Actor, Critic]
math: true
---

# 12강 — Policy Gradient / Actor-Critic 강의 정리본

---

## 0. 강의 한눈에 보기

**교수님의 강의 프레이밍**
지난 강의(DQN의 Experience Replay)를 간단히 정리하고, "이번 시간에는 액터 크리틱인데 복습이 필요하다"며 Policy Gradient 전체 맥락부터 다시 잡았다. "REINFORCE는 간단했는데, 액터 크리틱으로 넘어가니까 엄청 많습니다"라고 예고한 뒤 하나씩 쌓아가는 방식으로 진행.

**개념 목록 및 중요도**

| 개념 | 중요도 |
|---|---|
| Policy Gradient 복습 (Score Function, REINFORCE) | ★★ |
| Actor-Critic 구조 (Actor vs Critic) | ★★★ |
| Q Actor-Critic (QAC) | ★★ |
| 세미 그래디언트 (Semi-gradient) | ★★★ |
| 바이어스 문제 & Compatible Function Approximation | ★ |
| Advantage Function | ★★★ |
| Advantage Actor-Critic (A2C) | ★★★ |
| A2C 수학적 정당성 (Baseline 증명) | ★ |
| TD Actor-Critic (TD error = Advantage) | ★★★ |
| TD(λ) Actor-Critic & Eligibility Traces | ★★ |

**지난 강의와의 연결**
DQN의 두 문제(① 학습 불안정 → 타겟 네트워크, ② 코릴레이션 → Experience Replay)를 전 시간에 다룬 것을 간략히 복습하며 시작. 이번 강의의 Actor-Critic은 "Policy Gradient + DQN을 섞은 것"으로 소개.

**다음 시간 예고**
교수님이 강의 말미에 "다음 시간부터는 액터 크리틱의 다양한 방법론들"을 다룰 것이라 예고. 시험 기간 이후에 복귀하며 무주코(MuJoCo), 마이작 설치 영상을 올릴 것이라 언급.

---

## 1. Policy Gradient 복습 ★★

**교수님의 설명 흐름**

"액터 크리틱이 Policy Gradient와 DQL을 섞은 것이기 때문에 Policy Gradient 복습부터 하겠다"며 시작. 복습의 목적이 단순 회상이 아니라, 이후 크리틱이 왜 필요해지는지를 이해시키기 위한 포석임.

1. **학습의 지향점**: "모든 모델의 학습은 지향점이 있어야 한다. 무엇을 하면 잘했다, 못했다라는 목표가 있어야 학습의 방향이 정해진다."
2. **좋은 폴리시의 세 가지 정의**를 복습:
   - 시작 상태 $s_1$에서의 밸류가 가장 큰 폴리시 (조건: 모든 에피소드가 동일한 상태에서 출발)
   - 모든 스테이트에 대해 밸류가 가장 큰 폴리시 (스테이트 분포 고려)
   - 특정 스테이트와 액션의 리워드가 가장 큰 폴리시
3. **미분의 문제**: "학습하려면 미분해야 하고, 미분하면 기대값을 구할 수 없게 된다."
4. **Likelihood Ratio Trick**: 기대값을 구할 수 있는 형태로 변환 → 스코어 함수 도출

**핵심 정의 / 수식**

$$\pi_\theta(s, a) = \mathbb{P}[a \mid s, \theta]$$

Policy Gradient Theorem:

$$\nabla_\theta J(\theta) = \mathbb{E}_{\pi_\theta}[\nabla_\theta \log \pi_\theta(s, a) \, Q^\pi(s, a)]$$

REINFORCE (MC 방식):

$$\Delta\theta_t = \alpha \nabla_\theta \log \pi_\theta(s_t, a_t) \, v_t \quad (v_t = \text{return})$$

Score Function:

$$\nabla_\theta \pi_\theta(s, a) = \pi_\theta(s, a) \nabla_\theta \log \pi_\theta(s, a)$$

**교수님이 쓴 비유·예시·표현**

> "교수님 표현: 미분을 하려고 하니까 문제가 생겼죠. 뭐가 문제가 생겼을까요? 미분을 하고 나면 기대값을 구할 수가 없게 됐어요. 그래서 기대값을 구하는 폼으로 만들기 위해서 Likelihood Ratio Trick을 썼습니다."

**강조 포인트 & 중요도 근거**

★★ — 지난 강의 내용 복습이지만, 이후 크리틱 도입 필요성을 이해하기 위해 반드시 이 흐름을 짚고 넘어갔음. REINFORCE의 $v_t$가 return이라는 점을 슬라이드에서 빨간 동그라미로 강조.

**주의·함정**

- $v_t$는 리턴(return)이지 밸류(value)가 아님 — 슬라이드에 "return!" 이라고 필기 강조
- REINFORCE는 MC와 동일한 특성: **에피소드 끝까지 기다려야 함, Variance가 큼**
- 스코어 펑션의 핵심 수학: $\nabla_\theta \pi_\theta(s,a) = \pi_\theta(s,a) \cdot \nabla_\theta \log \pi_\theta(s,a)$ — 이게 왜 성립하느냐? $\nabla \log x = \frac{\nabla x}{x}$ 이므로 양변에 $\pi_\theta$를 곱하면 됨. 슬라이드 3에 필기로 "이렇게 나옴"이라고 화살표로 표시.

---

## 2. Actor-Critic 개요 — 왜 필요한가? ★★★

**교수님의 설명 흐름**

REINFORCE의 MC 방식이 variance가 크다는 한계에서 출발. "MC 말고 다른 방식을 써보자"라는 발상으로, $Q^\pi(s,a)$를 어떻게 얻을 것인지를 문제로 설정.

선택지는 두 가지:
1. 샘플에서 직접 리턴 계산 → REINFORCE (MC와 동일)
2. 샘플을 이용해 **추정** → 추정해주는 **별도 모델**이 필요

교수님 표현: "SA에 대한 밸류가 뭐니라고 물어봤을 때 답해 주는 애가 있어야 한다. 그게 테이블이든, 네트워크든."

그래서 두 네트워크가 등장:
- **액터 (Actor)**: 폴리시 네트워크 (파라미터 $\theta$) → 액션을 함
- **크리틱 (Critic)**: 밸류 네트워크 (파라미터 $w$) → 액션을 평가함

**핵심 정의 / 수식**

$$Q_w(s, a) \approx Q^{\pi_\theta}(s, a)$$

$$\nabla_\theta J(\theta) \approx \mathbb{E}_{\pi_\theta}[\nabla_\theta \log \pi_\theta(s, a) \, Q_w(s, a)]$$

$$\Delta\theta = \alpha \nabla_\theta \log \pi_\theta(s, a) \, Q_w(s, a)$$

- $\nabla_\theta \log \pi_\theta(s, a)$: Gradient Ascent의 **방향** 설정
- $Q_w(s, a)$: 업데이트의 **크기** 결정

**교수님이 쓴 비유·예시·표현**

> 교수님 표현 (위플래시 영화): "이 영화 혹시 보셨습니까? 위플래시입니다. 이 사람(교수)이 크리틱이고 이 사람(드러머)이 액터인 거든. 뭔가 크리틱은 나쁜 것 같은데, 그렇지 않습니다."

> 교수님 표현 (이름 어원): "액터는 결국 액션을 하기 때문에 액터인 거고, 그 액터에 대한 반대말을 그냥 크리틱이라고 이름을 붙인 게 아닌가 싶습니다. 실제도 맞고요. 액션에 대해서 평가를 하는 거니까요."

> 교수님 표현 (SLAM 비유): "이건 슬램이랑 좀 비슷해요. 슬램은 내가 위치를 정확히 알면 맵을 잘 만들 수 있고, 맵을 정확히 알면 위치를 정확하게 추정할 수 있는데, 둘 다 모르면 어렵다는 거잖아요. 이것도 마찬가지로, 좋은 액션이 있으면 밸류를 정확히 추정할 수 있고, 밸류가 정확하면 좋은 액션을 만들어낼 수 있지만, 한 가지가 망가지면 악순환이 생길 수 있다."

**강조 포인트 & 중요도 근거**

★★★ — 이 개념이 강의 전체의 뼈대. 슬라이드에서 "Q^π(s,a)를 구할 모델이 또 필요", "Critic Network"로 필기. 두 네트워크의 역할 분담($\theta$ vs $w$)을 슬라이드 곳곳에서 반복 강조. 교수님이 두 개의 비유(위플래시, SLAM)를 들었다는 것 자체가 이 개념을 핵심으로 여긴다는 신호.

---

## 3. Q Actor-Critic (QAC) 알고리즘 ★★

**교수님의 설명 흐름**

가장 단순한 Actor-Critic 구현. $Q_w$로 action-value를 추정하고, TD로 크리틱을 학습.

**핵심 정의 / 수식**

Algorithm 10.1: The simplest actor-critic algorithm (QAC)

```
초기화: s, θ (액터 네트워크 파라미터), w (밸류 네트워크 파라미터)

각 스텝마다:
  A_t ~ π_θ(s_t) → 액터가 액션 생성
  환경에서 R_{t+1}, S_{t+1} 수신
  δ = R_{t+1} + γ q(S_{t+1}, A_{t+1}, w) - q(S_t, A_t, w)   [TD 에러]

  Actor 업데이트: θ_{t+1} = θ_t + α_θ ∇_θ ln π_θ(A_t|S_t) q(S_t, A_t, w_t)
  Critic 업데이트: w_{t+1} = w_t + α_w [TD Error] ∇_w q(S_t, A_t, w_t)
```

**교수님이 쓴 비유·예시·표현**

> 교수님 표현: "δ가 뭔지 아시겠습니까? TD 네 글자입니다. TD 타겟 뺀 거죠. r + Q(s', a') - Q(s, a). 이거는 크리틱 학습이고, 이건 DQN이랑 같은 거예요."

> 교수님 표현: "SARS가 생기게 됩니다. S, A, Reward, next State. 그러면 한 번 업데이트를 할 수 있다는 거고요."

> 교수님 표현 (플러스 이유): "왜 더해주냐. 우리는 에러를 미니마이즈하는 게 목적인데, 로스가 -Q니까 미분하면 앞에 마이너스가 붙어 플러스가 된다."

**강조 포인트 & 중요도 근거**

★★ — 구체적 알고리즘을 보여주었지만, 이후 TD 크리틱으로 대체되므로 "출발점"으로서의 의미. 슬라이드에 "Q-AC는 Policy Network도, Value 네트워크도 필요하고, 또 하나의 네트워크가 사용된다"로 필기.

**주의·함정**

- 알고리즘 슬라이드(10, 11)에서 Actor 업데이트가 Critic 업데이트보다 먼저 나와 있으나, 교수님 말씀: "순서는 달라도 맞다. Critic 학습할 때 값이 바뀌는 건 아니니까."

---

## 4. 세미 그래디언트 (Semi-gradient) ★★★

**교수님의 설명 흐름**

QAC 알고리즘에서 크리틱 업데이트 규칙($w \leftarrow w + \beta\delta$)에 왜 플러스가 붙는지를 설명하다가, "왜 TD 타겟 부분을 미분하지 않느냐"는 문제로 확장. "이 질문을 다른 학기에도 많이 받았다"며 세미 그래디언트를 상세히 설명.

**핵심 정의 / 수식**

크리틱의 로스 함수:

$$L(w) = \frac{1}{2}\left(r + \gamma Q_w(s', a') - Q_w(s, a)\right)^2$$

- **Full gradient**: $Q_w(s', a')$와 $Q_w(s, a)$ 둘 다 $w$에 대해 미분
- **Semi-gradient**: $r + \gamma Q_w(s', a')$ 부분(TD 타겟)은 **상수 취급**, $Q_w(s, a)$만 미분

$$\nabla_w L(w) = -(r + \gamma Q_w(s', a') - Q_w(s, a)) \nabla_w Q_w(s, a) = -\delta$$

$$\therefore w \leftarrow w + \beta\delta$$

**교수님이 쓴 비유·예시·표현**

> 교수님 표현: "이 앞에 있는 거는 그냥 상수 취급을 하고 뒤에 있는 우리가 원하는 이 친구만 미분을 해요. 그게 세미 그래디언트라고 얘기를 합니다."

> 교수님 표현: "빨간색으로 두 줄 쳐놓은 이 친구는 값은 구하되 그 구한 값을 그냥 상수로 취급한다는 거예요. 학습할 때 얘네들로 역전파(backpropagation)를 하지 않는다는 거 이쪽으로 간다는 거예요."

> 교수님 표현: "코드에서는 detach라고 하나, 딱 끊는 거 있잖아요. 더 이상 얘는 학습한 애가 아니야라고 끊는 거 그렇게 하시면 돼요."

> 교수님 표현 (세미 그래디언트를 쓰는 이유 세 가지):
> 1. **안정성**: 풀 그래디언트를 쓰면 타겟이 계속 움직여 학습이 불안정하다
> 2. **벨만 방정식의 의미**: "벨만 이큐에이션 자체가 지향하는 것은 TD 타겟을 따라가는 거지 TD 타겟을 바꾸는 것이 아니에요"
> 3. **실용성**: 세미 그래디언트가 더 빠르고 잘 된다

**강조 포인트 & 중요도 근거**

★★★ — 슬라이드에 "Full gradient로 구하면 불안정한 학습", "값은 받아오되 상수로 취급", "미분할 때만 상수 취급" 등 다중 필기. 교수님이 학생 질문에 응답하며("저 값은 계속 바뀔 수 있지만 미분할 때만 상수 취급한다는…") 추가 확인. DQN 때도 동일하게 적용되는 원리이므로 반드시 이해해야 할 개념.

---

## 5. 바이어스 문제 & Compatible Function Approximation ★

**교수님의 설명 흐름**

$Q^\pi(s,a)$(폴리시 하에서의 실제 밸류) vs $Q_w(s,a)$(모델로 추정한 밸류)가 다른 것을 그냥 써도 되냐는 문제 제기. "그냥 모르니까 추정을 하겠다는 건데 그래도 되냐"는 질문에 "된다"라고 답.

**핵심 정의 / 수식**

두 조건을 만족하면 bias 없음:
1. **Compatible features**: 크리틱의 그래디언트가 스코어 함수와 일치
$$\nabla_w Q_w(s, a) = \nabla_\theta \log \pi_\theta(s, a)$$
2. 밸류 함수 파라미터가 MSE(TD 에러)를 최소화
$$w = \arg\min_w \mathbb{E}_{\pi_\theta}[(Q^\pi(s,a) - Q_w(s,a))^2]$$

핵심 인사이트:
- 폴리시 그래디언트는 **방향** $\nabla_\theta \log \pi_\theta$에만 의존함
- $Q_w$가 $Q^\pi$와 완전히 같을 필요는 없고, **그 방향으로 프로젝션된 값만 같으면** 됨 (내적 관계)

**교수님이 쓴 비유·예시·표현**

> 교수님 표현: "이해하는데 그렇게 중요한 건 아닙니다. 제가 강의하기 전에 이렇게 얘기하는 게 맞는지 모르겠는데... 이거를 Actor-Critic이 왜 되는지 그 정당성을 수학적으로 증명한 거라서 개념만 좀 이해해주셨으면 좋겠다."

> 교수님 표현: "이쪽 방향의 프로젝션된 값이 같으면 되는 거 안 되는 거예요. 그게 가능하면 얘랑 얘랑은 다르지만 결국 학습했을 때 옵티멀로 갈 것이다라고 할 수 있다는 거죠."

**강조 포인트 & 중요도 근거**

★ — 교수님이 **직접 "그렇게 중요한 건 아닙니다"라고 명시**했음. 슬라이드에 "Not Important" 필기. Actor-Critic의 이론적 정당성을 제공하지만 실제 구현에는 반드시 필요하지 않음.

---

## 6. Advantage Function ★★★

**교수님의 설명 흐름**

QAC에서 Q값을 그대로 쓰는 것보다 더 좋은 방법을 제안: "Advantage를 쓰면 학습이 훨씬 잘 된다." Advantage의 직관적 의미부터 시작.

**핵심 정의 / 수식**

$$A(s, a) = Q_w(s, a) - V_v(s)$$

- $V^{\pi_\theta}(s)$: 스테이트 밸류 (그 스테이트에서 가능한 모든 액션의 기대값)
- $Q^{\pi_\theta}(s, a)$: 액션 밸류
- $A(s, a)$: 해당 액션이 평균 대비 얼마나 좋은지

모든 액션의 Advantage 합 = 0 (평균에서 뺀 값이므로)

**교수님이 쓴 비유·예시·표현**

> 교수님 표현 (시험 비유): "우리가 다 시험을 쳤어. 여러분들 시험 평균이 몇 점입니다. 이러면 어떤 사람은 평균보다 몇 점이 높을 거고 어떤 사람은 평균보다 몇 점이 낮겠죠. 그 개념이라고 보시면 돼요."

> 교수님 표현 (학습이 잘 되는 이유): "Q값이 19,800, 1000, 900, 800이라고 해볼게요. 그러면 세 방향 다 가는데, '더 가냐 덜 가냐'의 문제가 되는 거예요. 근데 Advantage를 쓰면 1100, 0, -100이 되니까, 가장 좋은 방향은 가는데, 가장 안 좋은 방향은 **반대로 가요**. 그래서 Advantage의 평균은 0이기 때문에 훨씬 더 학습에 유리하다고 얘기를 할 수 있다."

**강조 포인트 & 중요도 근거**

★★★ — "미리 말씀을 드리면 우리가 사용하는 액터 크리틱은 **다 Advantage Actor-Critic**이에요"라고 직접 선언. Q Actor-Critic은 거의 사용하지 않는다고 명시. 슬라이드에 "대부분 이거", "Advantage써서 학습하겠다 / 더 잘됨" 필기.

---

## 7. Advantage Actor-Critic (A2C) & 구현 문제 ★★★

**교수님의 설명 흐름**

Advantage를 쓰는 A2C를 소개하되, 두 가지 문제를 지적:
1. 수학적으로 맞냐? (=$V_v(s)$를 baseline으로 빼도 되냐?)
2. 구현 문제 (네트워크 3개 필요)

이 두 문제를 설명한 뒤 "그래서 별로 안 좋다"고 하며 다음 단계(TD Actor-Critic)로 넘어가는 구조.

**핵심 정의 / 수식**

$$\nabla_\theta J(\theta) = \nabla_\theta \mathbb{E}_{\pi_\theta}[\nabla_\theta \log \pi_\theta(s, a) \, A(s, a)]$$

$$\nabla_\theta J(\theta) = \nabla_\theta \mathbb{E}_{\pi_\theta}[\nabla_\theta \log \pi_\theta(s, a) \, (Q_w(s, a) - V_v(s))]$$

A2C에서 필요한 네트워크:
- $\pi_\theta$: 폴리시 네트워크
- $Q_w(s, a)$: Q 밸류 네트워크
- $V_v(s)$: 스테이트 밸류 네트워크

**$V_v(s)$를 빼도 되는 이유 (Baseline 증명):**

$$\mathbb{E}_{\pi_\theta}[\nabla_\theta \log \pi_\theta(s, a) \, V_v(s)] = 0$$

왜냐하면 $V_v(s)$는 $a$에 의존하지 않으므로 $\sum_a \nabla_\theta \pi_\theta(s, a) = \nabla_\theta \sum_a \pi_\theta(s, a) = \nabla_\theta 1 = 0$

따라서 $V_v(s)$를 빼도 **기댓값은 동일** (Bias 없음)

**교수님이 쓴 비유·예시·표현**

> 교수님 표현 (A2C의 모순): "제가 아까 '우리가 사용하는 액터 크리틱은 다 Advantage Actor-Critic이다'라고 얘기를 했는데, 이거 별로 안 좋다라고 얘기를 하면은 모순이 되죠. 그렇지만 그건 사실입니다. 그래서 다음 걸로 넘어갑니다."

> 교수님 표현 (네트워크 3개 문제): "Q와 V라고 하는 것이 엄밀히 말하면 Q를 평균낸 게 V여야 되는데, 각자 따로 학습하면 서로 어떤 값인지 모르잖아요. 원리적으로는 Advantage를 쓰는 것이 좋은데 이런 구조로 학습하면 학습이 잘 되지 않아요."

**강조 포인트 & 중요도 근거**

★★★ — "대부분 다 Advantage Actor-Critic이다"라는 선언, 슬라이드에 "대부분 이게"로 필기. 단, 직접 구현(3 networks)은 문제가 있어 TD로 대체함.

---

## 8. TD Actor-Critic (TD Error = Advantage) ★★★

**교수님의 설명 흐름**

A2C의 구현 문제(네트워크 3개)를 해결하는 핵심 아이디어: **TD 에러 자체가 Advantage의 불편 추정량**임을 보여줌. 이 부분이 교수님이 가장 강조한 개념.

**핵심 정의 / 수식**

**스테이트 밸류의 TD 에러:**

$$\delta^{\pi_\theta} = r + \gamma V^{\pi_\theta}(s') - V^{\pi_\theta}(s)$$

**TD 에러의 기댓값 = Advantage임을 증명:**

$$\mathbb{E}_{\pi_\theta}[\delta^{\pi_\theta} \mid s, a] = \mathbb{E}_{\pi_\theta}[r + \gamma V^{\pi_\theta}(s') \mid s, a] - V^{\pi_\theta}(s)$$
$$= Q^{\pi_\theta}(s, a) - V^{\pi_\theta}(s)$$
$$= A^{\pi_\theta}(s, a)$$

왜냐하면:
- $s$와 $a$가 주어져도 $s'$은 확률적으로 달라질 수 있음 → 여전히 기댓값 필요
- $r + \gamma \mathbb{E}[V(s')] = Q(s, a)$ (정의에 의해)

따라서 실제 TD 에러(샘플)를 Advantage 대신 써도 됨!

$$\delta_v = r + \gamma V_v(s') - V_v(s) \quad (\text{approximate TD error})$$

**TD Actor-Critic 알고리즘 (One-Step):**

```
초기화: s, θ

각 스텝마다:
  A_t ~ π_θ(S_t)
  환경에서 R_{t+1}, S_{t+1} 수신
  δ_t = R_{t+1} + γ v_w(S_{t+1}) - v_w(S_t)   [TD 에러 = Advantage]
  
  Critic: w ← w + β δ_t ∇_w v_w(S_t)           [TD(0)]
  Actor:  θ ← θ + α δ_t ∇_θ log π_θ(A_t | S_t) [Policy gradient]
```

**교수님이 쓴 비유·예시·표현**

> 교수님 표현 (핵심 결론): "s가 주어지고 a가 주어졌다라고 해서 next state가 고정된 건 아니거든요. 확률에 따라서 달라질 수 있다는 거. 거기 다양한 next state에 대한 기대값이 TD 타겟이 되는 거고... 결국에는 이게 Q가 되는 거예요. Q에서 V를 빼니까 Advantage가 됩니다."

> 교수님 표현 (V 네트워크 하나로 OK): "이걸 계산해서 Advantage를 쓰면 장점이 뭘까요? **모델 하나를 더 쓸 필요가 없어요.** 우리는 V 네트워크만 쓰면 된다는 거예요."

> 교수님 표현: "어떻게 보면은 우리가 지금까지 본 것 중에서 제일 간단합니다."

> 교수님 표현 (최종 정리): "TD 크리틱이라고 하지만 실제로 여기에 이 델타라고 하는 것이 Advantage라고 말씀드렸잖아요. 그래서 우리가 보통 TD Actor-Critic이라고 하면 얘를 얘기합니다. 이게 우리가 일반적으로 얘기하는 Advantage Actor-Critic이라고 얘기를 합니다."

**강조 포인트 & 중요도 근거**

★★★ — 이 강의의 결론. "TD 크리틱 = 실제 Advantage Actor-Critic"이라는 등호 관계를 마지막에 강조하며 확인. 슬라이드 필기에 "나 대신 TD ERROR로 쓴다", "네트워크 하나", "제일 간단" 등 다중 강조.

**주의·함정**

- A2C(어드밴티지 액터 크리틱)라는 이름이 붙은 구현들은 거의 대부분 실제로는 **TD 액터 크리틱 방식** (V 네트워크 하나만 사용)
- Q 액터 크리틱과 헷갈리지 말 것: Q-AC는 Q 네트워크, TD-AC는 V 네트워크만 사용

---

## 9. TD(λ) Actor-Critic & Eligibility Traces ★★

**교수님의 설명 흐름**

TD를 쓰면 n-step TD, TD(λ)의 개념이 그대로 적용됨을 설명. "TD가 나오면 반드시 나와야 되는 게 있죠. TD의 자식들."

**핵심 정의 / 수식**

**n-step Advantage:**

$$\hat{A}_t^{(n)} = r_{t+1} + \gamma r_{t+2} + \ldots + \gamma^{n-1} r_{t+n} + \gamma^n V(s_{t+n}) - V(s_t)$$

**Forward-view TD(λ) (λ-가중합):**

$$\Delta\theta = \alpha(v_t^\lambda - V_v(s_t)) \nabla_\theta \log \pi_\theta(s_t, a_t)$$

($v_t^\lambda$: n-step 밸류의 λ 가중합)

**Backward-view TD(λ) (Eligibility Traces):**

$$\delta_t = r_{t+1} + \gamma V_v(s_{t+1}) - V_v(s_t)$$

$$e_{t+1}(s) = \begin{cases} \gamma\lambda e_t(s) + \nabla_\theta \log \pi_\theta(s_t, a_t) & \text{if } s = s_t \\ \gamma\lambda e_t(s) & \text{otherwise} \end{cases}$$

$$\Delta\theta = \alpha \delta_t e_t$$

TD(λ)에서와의 차이: 현재 스테이트 방문 시 eligibility에 **1을 더하는 것이 아니라** $\nabla_\theta \log \pi_\theta(s_t, a_t)$를 더함.

**교수님이 쓴 비유·예시·표현**

> 교수님 표현 (Eligibility Traces 차이): "예전에 TD에서는 현재 방문한 스테이트는 1을 더해주고 나머지는 깎았는데, 여기서는 이상한 값을 더해주는 거야. 이상한 값을 더해주는 이유는 기울기를 정해준다고. 쉽게 얘기하면, x축 y축 z축이 있다고 했을 때 현재의 폴리시 그래디언트 방향은 x축이에요. 그러면 지나온 애들 중에서도 x축에 대해서만 업데이트를 준다는 겁니다."

> 교수님 표현: "어떤 애는 y축으로만 업데이트를 해준 애가 있다면, 이번에 얻은 TD Advantage는 걔한테 적용이 안 되겠죠. 내가 이 그래디언트에 연관된 애들만 업데이트해준다는 거예요."

**강조 포인트 & 중요도 근거**

★★ — "TD가 나오면 n-step과 TD(λ)가 반드시 따라온다"는 패턴을 강조. 단, Eligibility Traces의 백워드뷰는 약간 복잡한 개념으로, 교수님도 "조금 이제 뭔가 너무 똑같으면 너무 똑같진 않겠죠"라며 차이점을 짚어줌.

---

## 10. Policy Gradient 전체 요약 ★★★

**교수님의 설명 흐름**

강의 마지막에 전체를 한 슬라이드로 정리. "가중치로 뭘 쓰느냐"에 따라 알고리즘이 달라지는 구조.

**핵심 정의 / 수식**

| 수식 | 알고리즘 |
|---|---|
| $\nabla_\theta J(\theta) = \mathbb{E}_{\pi_\theta}[\nabla_\theta \log \pi_\theta(s,a) \, v_t]$ | REINFORCE |
| $= \mathbb{E}_{\pi_\theta}[\nabla_\theta \log \pi_\theta(s,a) \, Q^w(s,a)]$ | Q Actor-Critic |
| $= \mathbb{E}_{\pi_\theta}[\nabla_\theta \log \pi_\theta(s,a) \, A^w(s,a)]$ | Advantage Actor-Critic |
| $= \mathbb{E}_{\pi_\theta}[\nabla_\theta \log \pi_\theta(s,a) \, \delta]$ | TD Actor-Critic |
| $= \mathbb{E}_{\pi_\theta}[\nabla_\theta \log \pi_\theta(s,a) \, \delta e]$ | TD(λ) Actor-Critic |

> 슬라이드 필기: "TD Actor-Critic과 TD(λ) Actor-Critic → **Advantage Actor-Critic**"

**교수님이 쓴 비유·예시·표현**

> 교수님 표현 (최종 정리): "결국에는 우리가 폴리시를 업데이트할 때 그 그래디언트 방향은 정해졌는데 가중치를 어떻게 정해줄 것이냐라고 했을 때, REINFORCE는 실제 리턴값, Q 크리틱은 Q 네트워크로 추정한 값, TD 크리틱은 델타(= Advantage)로. 그래서 우리가 보통 TD 크리틱이라고 하는 게 일반적으로 얘기하는 Advantage Actor-Critic입니다."

**강조 포인트 & 중요도 근거**

★★★ — 이 표가 강의 전체의 요약이자 핵심. "Most implementations commonly referred to as 'A2C' are actually built as TD actor-critic variants (using only a v network)"를 슬라이드에 명시.

---

## 마지막 점검

**강의자료와 전사본이 어긋난 부분**
- 특별한 충돌 없음. 전사본이 슬라이드 순서를 대체로 따르되, 슬라이드 9(Appendix: w 업데이트 규칙)를 QAC 알고리즘(슬라이드 10) 이전에 먼저 설명하는 순서로 진행.

**전사가 불명확해 추정한 부분**
- "라이클리 로드 트립" → **Likelihood Ratio Trick** [전사 오류]
- "리인폴스" / "리인포스먼트" → **REINFORCE** [전사 오류]
- "TV" / "TDL라" → **TD 에러** [전사 오류]
- "컴페터블 펑션 어프록시메이션" → **Compatible Function Approximation** [전사 오류]
- "디큐엔" → **DQN** [전사 오류]
- "바이러스" → **Bias (바이어스)** [전사 오류]
- "어드밴티지" / "어드벤테이션" / "어드밴티지" → **Advantage** [전사 혼용]
- "앨리저빌리티 트레이스" / "엘리제리미티 트레이스" → **Eligibility Trace** [전사 오류]

**교수님이 남긴 예고**
- 다음 시간: "액터 크리틱의 다양한 방법론들" (DQN 이후 개선 방향들)
- 시험 기간 이후 복귀 전: MuJoCo / 마이작 설치 녹화 강의 업로드 예정
- 교수님 코멘트: "DQN이 네이처에 실렸다라고 하면 세상에 수많은 연구자들이 그걸 물어뜯으려고 달려들지 않겠습니까. DQN이 다 좋은데 이게 문제야, 이걸 해결하기 위해서 이렇게 하면 돼, 이런 식으로 진행할 거예요."