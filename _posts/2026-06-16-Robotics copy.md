---
title: "Robotics Dynamics"
description: "Robotics"
author: KEYBEOMM
date: 2026-06-16 00:00:00 +0900
categories: [Robotics]
tags: [robot, dynamics]
math: true
---

# 06/11 로봇공학 강의 정리본 — Dynamics of Robot Manipulators (로봇 매니퓰레이터 동역학)

> 강의자료: `260611_로봇공학.pdf` (56쪽) · 전사본: `06_11_dynamics_robot_manipulator.md`
> 교수: 김종우 · **시험 전 마지막 강의** (교수님 명시: "시험 전 마지막 시간이니까 더 이상 진도는…") → 기말고사 직결 내용

---

## 0. 강의 한눈에 보기

### 슬라이드 Outline (PDF p2/9/21/34/44 — 목차 슬라이드가 반복되며 섹션을 알려줌)
**Dynamics of Robot Manipulators**
1. **Introduction** (p3–8)
2. **Euler–Lagrange Formulation** (p10–20)
3. **Kinetic and Potential Energy of a Robot** (p22–33)
4. **Robot Dynamics using Euler–Lagrange** (p35–43)
5. **Examples of Dynamic Models** (p45–55)

### 교수님이 도입부에서 잡은 큰 그림
- "다이나믹스는 **세컨드 오더(2차) 익스프레션**이 들어간다" — 가속도·각가속도가 등장하기 때문. 이게 이 강의 전체를 관통하는 첫 문장이다.
- 키네매틱스(지난 강의들)에서는 무시했던 **자중(self-weight)·관성·코리올리**가 동역학에서는 전부 살아난다 → 그래서 **동역학을 맨 마지막에** 다룬다("고려할 식들이 선발적으로 많다 보니까").
- 이번 시간 흐름: (지난 시간 복습: 에너지·COM·관성텐서) → **로봇 동역학 표준형** 다시 확인 → **Euler–Lagrange 항별(term-by-term) 유도 + Christoffel 기호** → **Generalized force 구성요소** → **5단계 계산 절차** → **2R 로봇 예제 완전 풀이**.

### 다룬 개념 + 중요도 한눈표
| # | 개념 | 슬라이드 | 등급 |
|---|---|---|---|
| 1 | 로봇 동역학 표준형 $M\ddot q + C\dot q + g = \tau$ | p5, p37 | ★★★ |
| 2 | Forward vs Inverse Dynamics | p6–7 | ★★ |
| 3 | 계산법: Euler–Lagrange vs Newton–Euler | p8 | ★★ |
| 4 | Euler–Lagrange 식 ($\mathcal L=T-U$, E-L 방정식) | p10–11 | ★★★ |
| 5 | 예제: Point mass / **Simple pendulum** | p12–15 | ★★★ |
| 6 | Exercise: Double pendulum | p16–18 | ★★ |
| 7 | Center of Mass (COM) | p22–24 | ★★ |
| 8 | Inertia Tensor (관성텐서) | p28–30 | ★★ |
| 9 | 로봇의 Kinetic/Potential Energy, **Mass matrix $M(q)$** | p31–33 | ★★★ |
| 10 | **항별 유도 + Christoffel 기호** | p35–43 | ★★★ |
| 11 | Generalized Force 구성 (모터관성·마찰·외력) | p38 | ★★ |
| 12 | **5단계 계산 절차** | p43 | ★★★ |
| 13 | **Example 1: 2R 매니퓰레이터 완전 풀이** | p45–55 | ★★★ |

### 지난/다음 강의 연결
- **지난 시간**: 로봇의 키네틱·포텐셜 에너지 유도, COM, 관성텐서 (이번 강의 앞부분에서 복습). 그 전 강의들에서 prismatic/revolute joint의 Jacobian 유도.
- **다음(=숙제·시험)**: "예제들을 통해서 어떻게 실제 (값을) …" 예고. **두 번째 예제 풀이를 캠퍼스에 업로드** → "기말고사 때 도움이 될 것." 강의 진도는 여기서 종료, 마지막에 연구실 소개(여담).

---

## 용어 대조표

| 영어 (슬라이드) | 한국어/구어 (전사본 발음) | 한 줄 뜻 |
|---|---|---|
| Dynamics | 다이나믹스 | 힘·토크가 운동에 미치는 영향(=운동의 원인) 연구, 2차 표현 |
| Generalized force | 제너럴라이즈드 폴스/홀스 | 관절 토크 + 외력/외부모멘트를 총칭하는 일반화 힘 |
| Forward dynamics | 포워드 다이나믹스 | $\tau$ 주어짐 → 운동 $\ddot q$ 구함 (시뮬레이션용) |
| Inverse dynamics | 인버스 다이나믹스 | 운동 $\ddot q$ 주어짐 → 필요한 $\tau$ 구함 (제어용) |
| Euler–Lagrange | 오일러 라그랑지 | 에너지(운동+위치) 기반 동역학 유도법, symbolic |
| Newton–Euler | 뉴턴 오일러 (전사: 비텀/위턴 오일러) | 힘·모멘트 평형 기반, recursive·실시간용 |
| Kinetic energy | 키네틱 에너지 (전사: 피네틱/티네틱) | 운동에너지 $T=\frac12 m\lVert\dot x\rVert^2$ |
| Potential energy | 포텐셜 에너지 | 위치에너지 $U=mgh$ |
| Lagrangian | 라그랑지안 | $\mathcal L = T - U$ |
| Mass / Inertia matrix | 메스/이너셜 매트릭스 | $M(q)$, 가속도에 곱해지는 관성행렬 (대칭·양정부호) |
| Coriolis force | 코리올리(코리올릭) | 회전계에서 두 관절 속도곱($\dot q_j\dot q_k$)에 걸리는 힘 |
| Centrifugal force | 센트리퓨걸/원심력 | 한 관절 속도제곱($\dot q_j^2$)에 걸리는 힘 |
| Inertia tensor | 이너셔 텐서 (전사: 인열차/이널샤) | 회전에서 "질량 역할"을 하는 3×3 행렬 |
| Center of Mass (CoM) | 센터 오브 매스 | 전체 질량이 집중·외력이 작용한다고 보는 점 |
| Christoffel symbols | 크리스토펠 기호 (전사: 크리스토 토벨) | $C$행렬 성분을 $M$의 편미분으로 표현하는 식 |
| Skew-symmetric | 스큐 시메트릭 (전사: 스키) | $\dot M - 2C$가 갖는 성질 |
| Positive definite | 포지티브 데피닛 (전사: 디퓨넷) | $M(q)\succ0$ |
| Coulomb / viscous friction | 쿨롱 / 비스코스 마찰 | 정지마찰 + 점성마찰 |
| Self-weight | 셀프 웨이트 | 링크 자중(중력항의 원인) |
| 2R / RR robot | 투알/2알 로봇 | revolute 2개 평면 매니퓰레이터 (시험 예제) |

---

## 1. 로봇 동역학과 표준형 $M\ddot q+C\dot q+g=\tau$ (Robot Dynamics) ★★★

**교수님의 설명 흐름** (p3–5)
- p3에서 **동역학의 정의**부터: "Why does it move? — 운동의 *원인*(forces/torques)을 다룬다." 가장 기초 예로 두 식을 나란히 둔다.
  - Newton 2법칙: $f = m\ddot x$ (병진)
  - Euler 2법칙: $\tau = J\ddot\theta$ (회전)
  - → "둘 다 **2차 미분(가속도)**이 들어간다. 그래서 다이나믹스는 항상 second-order." (전사 시작부의 핵심 메시지)
- p4 **Robot Dynamics**: 로봇에서는 **일반화 힘 $\tau(t)$ ↔ 로봇 운동 $q(t),\dot q(t),\ddot q(t)$** 의 관계가 관심사. 일반화 힘은 두 종류:
  - $\tau_{articular}$: 관절의 (모터) 토크
  - $\tau_{externals}$: 환경이 로봇에/로봇이 환경에 가하는 외력·외부토크 (엔드이펙터·링크 등 어디든 작용 가능)
  - 운동은 **joint trajectory** 또는 **Cartesian trajectory**(엔드이펙터).

**핵심 정의 / 수식** (p5 — 이 강의의 결론을 미리 보여주는 슬라이드)
$$\boxed{\,M(\mathbf q)\,\ddot{\mathbf q} + C(\mathbf q,\dot{\mathbf q})\,\dot{\mathbf q} + \mathbf g(\mathbf q) = \boldsymbol\tau\,}$$
- $\mathbf q$: generalized joint coordinates
- $M(\mathbf q)$: **mass/inertia matrix** — 가속도 $\ddot q$에 걸리는 항
- $C(\mathbf q,\dot{\mathbf q})$: **centrifugal & Coriolis** — 속도 $\dot q$에 걸리는 항
- $\mathbf g(\mathbf q)$: **gravity** — $\ddot q,\dot q$가 안 들어가고 $q$만 들어감
- $\boldsymbol\tau$: generalized force
- "동역학 모델링의 목표 = **$M,C,g$를 찾는 것**. 이걸 알면 제어와 시뮬레이션이 가능하다."

**교수님이 쓴 비유·표현**
- 세 항의 의미를 전사에서 반복: "메스에 의한(가속도) 항 + **회전계라서 붙는 코리올리 항**($\dot q$ 곱) + 자중에 의한 그래비티 항."
- **코리올리 직관(★ 기억할 비유)**: "회전목마(빅휠)에서 마주보고 공을 던지면, 바깥 관찰자에겐 직선인데 **회전계 안 사람에겐 휘어 보인다**. 로봇 매니퓰레이터는 결국 revolute joint들의 조합 = 회전계의 연쇄라서 코리올리를 반드시 고려해야 한다." → "Craig 책, 공학수학에 코리올리 예제 있다"고 환기.

**⭐ 시험 대비**
- **재현 목표**: 임의 로봇의 동역학을 항상 이 박스형으로 정리할 수 있어야 한다. 어떤 항이 $M$/$C$/$g$인지 즉시 구분.
- **예상 유형**: 개념비교형(세 항의 물리적 의미) + 모든 유도형 문제의 최종 정리틀.
- **핵심 구분 암기**: $\ddot q$항=관성, $\dot q$항=원심+코리올리, $q$만=중력. **중력항엔 $\dot q,\ddot q$가 절대 안 들어간다.**

---

## 2. Forward Dynamics vs Inverse Dynamics ★★

**설명 흐름** (p6–7) — "초록색(주어짐) → 빨간색(구함)" 으로 방향을 강조.

| | **Forward Dynamics** (p6) | **Inverse Dynamics** (p7) |
|---|---|---|
| 주어짐 → 구함 | $\boldsymbol\tau(t)$ → $\ddot{\mathbf q}(t)$ | $\mathbf q_d,\dot{\mathbf q}_d,\ddot{\mathbf q}_d$ (desired) → $\boldsymbol\tau_d(t)$ |
| 의미 | 토크 넣으면 어떻게 움직이나 | 원하는 운동 내려면 토크 얼마? |
| 주 용도 | **dynamic simulation** | **control (제어)** |
| 처리 | direct dynamic model → **numerical integration** → $q$ | 가속도 레벨에서 제어법칙으로 $\tau$ 해석적 계산 |

**교수님 표현**: "실제 실험 안 하고 시뮬레이션에서 $\tau$ 주고 모션 뽑을 때가 forward. 반대로 '엔드이펙터를 2 m/s로 접근시키고 1.5 m/s로 옮긴다'는 **desired motion**이 있을 때 필요한 토크를 구하는 게 inverse = 제어에서 쓰는 값."

**⭐ 시험 대비**: 개념비교형. "forward=시뮬레이션, inverse=제어"와 입출력 방향을 헷갈리지 말 것.

---

## 3. 계산 방법: Euler–Lagrange vs Newton–Euler ★★

**설명 흐름** (p8) — "동역학(2학년 동역학 시간에 배웠죠)에 두 가지 접근이 있다."

| | **Euler–Lagrange** | **Newton–Euler** |
|---|---|---|
| 기반 | **에너지** (kinetic $T$, potential $U$) | **힘·토크 평형** (forces $f$, moments $m$) |
| 식 형태 | **symbolic** (기호적) | **numerical, recursive** |
| 적합 용도 | 동역학 특성 분석, 제어기 설계 분석 | **실시간 inverse dynamics**, 제어기 구현 |

- 슬라이드 note: E-L은 d'Alembert 원리, Hamilton 원리, 가상일 원리 등에서 유도 가능.
- **교수님 표현**: "에너지 기반(E-L)은 동역학 **속성/제어 분석**에 많이 쓰고, **뉴턴 오일러**는 **리얼타임**에 맞게 빠르게 연산되어 실시간 inverse dynamics에 많이 쓴다. 이번 강의는 **왼쪽(E-L)** 방식으로 접근한다."

**⭐ 시험 대비**: 핵심 구분 암기 — **E-L=에너지·분석·symbolic / NE=힘평형·실시간·recursive**. "어느 쪽이 실시간 제어 구현에 유리한가?"류 비교 문제 대비.

---

## 4. Euler–Lagrange Formulation 기본 ★★★

**설명 흐름** (p10–11)

**핵심 정의 / 수식** (p10)
- 운동에너지(강체): $\displaystyle T=\frac12 m\lVert\dot{\mathbf x}\rVert^2$ (표기: $\mathcal T,T,K,E_c$)
- 위치에너지: $U=mgh$ (표기: $\mathcal U,U,P,E_p$)
- **Lagrangian**: $\displaystyle \boxed{\mathcal L(\mathbf q,\dot{\mathbf q}) = T(\mathbf q,\dot{\mathbf q}) - U(\mathbf q)}$ ,  $\mathbf q\in\mathbb R^n$ generalized coordinate(보통 joint variable)

**Equations of Motion** (p11) — 두 형태를 반드시 같이 외울 것
- 항별(term by term): $\displaystyle \boxed{\frac{d}{dt}\!\left(\frac{\partial \mathcal L}{\partial \dot q_i}\right) - \frac{\partial \mathcal L}{\partial q_i} = \tau_i},\quad i=1,\dots,n$
- 벡터형(vector form): $\displaystyle \frac{d}{dt}\!\left(\frac{\partial \mathcal L}{\partial \dot{\mathbf q}}\right)^{\!T} - \left(\frac{\partial \mathcal L}{\partial \mathbf q}\right)^{\!T} = \boldsymbol\tau$
- 우변 $\tau_i$ = $q_i$에 대해 일(work)을 하는 generalized force.

**⭐ 시험 대비**
- **재현 목표**: $T,U$만 세울 수 있으면 → $\mathcal L=T-U$ → E-L 방정식에 대입 → 운동방정식. **이 4단계 루틴을 손으로.**
- **예상 유형**: 유도형(거의 모든 문제의 출발점).
- **자주 틀리는 지점**: ① $\frac{d}{dt}$(시간 전미분)와 $\frac{\partial}{\partial q_i}$(편미분) 구분, ② 부호(`-`가 $\partial\mathcal L/\partial q_i$ 앞에 붙음), ③ $q$와 $\dot q$를 **독립변수**로 보고 편미분.

---

## 5. 예제: Point Mass & Simple Pendulum ★★★

> 교수님이 칠판/슬라이드에서 **직접 전개**한 두 예제. 단순펜듈럼은 시험에 그대로 또는 2R 로봇의 축소판으로 나올 1순위.

### 5-1. Example 1: Point mass (p12)
질량 $m$이 $y$방향으로만 움직이고 위로 $f$, 아래로 $mg$.
- (a) **Newton**: $\sum f_i = m\ddot y \Rightarrow f - mg = m\ddot y \Rightarrow \boxed{m\ddot y + mg = f}$
- (b) **Euler–Lagrange**:
  - $T=\tfrac12 m\dot y^2$, $\quad U=mgy$, $\quad \mathcal L = T-U = \tfrac12 m\dot y^2 - mgy$
  - $\frac{d}{dt}\!\big(\tfrac{\partial\mathcal L}{\partial\dot y}\big) - \tfrac{\partial\mathcal L}{\partial y} = f \Rightarrow \frac{d}{dt}(m\dot y) + mg = f \Rightarrow \boxed{m\ddot y + mg = f}$
  - → "두 방법이 **같은 답**을 준다"는 걸 보여주는 게 포인트.

### 5-2. Example 2: Simple Pendulum (p13–14) ★★★
좌표: pivot 원점, $x$ 오른쪽, $y$ 위쪽. 각 $\theta$, 길이 $l$, 질량 $m$, 토크 $\tau$.

**(1) 위치·속도** (p13)
$$x = l\sin\theta,\quad y = -l\cos\theta \;\Rightarrow\; \dot x = \dot\theta\, l\cos\theta,\quad \dot y = \dot\theta\, l\sin\theta$$
**(2) 에너지**
$$U = mg(-l\cos\theta) = -mgl\cos\theta$$
$$T = \tfrac12 m\lVert\dot{\mathbf x}\rVert^2 = \tfrac12 m(\dot x^2+\dot y^2) = \tfrac12 m\big(\dot\theta^2 l^2\cos^2\theta + \dot\theta^2 l^2\sin^2\theta\big) = \tfrac12 m l^2\dot\theta^2$$
**(3) Lagrangian** : $\mathcal L = T-U = \tfrac12 ml^2\dot\theta^2 + mgl\cos\theta$

**(4) 미분 & E-L** (p14)
$$\frac{\partial\mathcal L}{\partial\theta} = -mgl\sin\theta,\qquad \frac{\partial\mathcal L}{\partial\dot\theta} = ml^2\dot\theta \;\xrightarrow{\;d/dt\;}\; ml^2\ddot\theta$$
$$\frac{d}{dt}\!\left(\frac{\partial\mathcal L}{\partial\dot\theta}\right) - \frac{\partial\mathcal L}{\partial\theta} = \tau \;\Rightarrow\; \boxed{ml^2\ddot\theta + mgl\sin\theta = \tau}$$
- **구조(Structure)**: $M(\theta)\ddot\theta + G(\theta) = \tau$ — 표준형의 1자유도판. (펜듈럼엔 코리올리·원심력 항이 없음에 주의)

### 5-3. Important note — 위치에너지 기준점 (p15) ★ 함정 주의
- $U$의 0점은 **임의로** 잡을 수 있다(편의대로). 같은 펜듈럼이라도:
  - pivot 기준: $U = mg(-l\cos\theta)$
  - 최저점 기준: $U = mg(l - l\cos\theta) = mgl(1-\cos\theta)$
- **중요한 건 $\dfrac{\partial U}{\partial\theta}$ 인데, 두 경우 모두 같다** → 운동방정식은 동일. "기준점은 편의에 맞춰 정하면 된다."

**⭐ 시험 대비**
- **재현 목표**: 단순펜듈럼 동역학을 좌표설정→$x,y$→$\dot x,\dot y$→$T,U$→$\mathcal L$→E-L까지 **빈손으로 완주**.
- **예상 유형**: 유도형(거의 확정). point mass처럼 Newton과 E-L 두 방법으로 같은 답 보이기도 출제 가능.
- **자주 틀리는 지점**: ① $y=-l\cos\theta$의 부호, ② $\dot y = \dot\theta l\sin\theta$ (음수 두 번으로 +), ③ $\cos^2+\sin^2=1$로 $T$가 깔끔히 $\tfrac12 ml^2\dot\theta^2$ 됨, ④ 기준점 함정.

---

## 6. Exercise: Double Pendulum (이중진자) ★★

**설명 흐름** (p16–18) — "이중진자 = revolute 2개가 시리얼로 연결 → **2-link 매니퓰레이터와 사실상 같은 형상**." 교수님은 "앞이랑 똑같은 방식이니 **풀어서 확인해 보라**"고 숙제로 남김(슬라이드엔 답 제공).

좌표: $\theta_1$(연직축 기준 1번 진자), $\theta_2$(연직축 기준 2번 진자), 약식 표기 $c_1=\cos\theta_1$, $s_{1-2}=\sin(\theta_1-\theta_2)$ 등.

**에너지** (p17, 고해상 슬라이드)
$$\mathcal T = \tfrac12 m_1 l_1^2\dot\theta_1^2 + \tfrac12 m_2\big(l_1^2\dot\theta_1^2 + l_2^2\dot\theta_2^2 + 2l_1 l_2\dot\theta_1\dot\theta_2\, c_{1-2}\big)$$
$$\mathcal U = -(m_1+m_2)g l_1 c_1 - m_2 g l_2 c_2$$

**운동방정식 (Answer, p16)** — 슬라이드 p17의 항별 미분과 일치하도록 차원 보정해 표기:
$$(m_1+m_2)l_1^2\ddot\theta_1 + m_2 l_1 l_2\ddot\theta_2 c_{1-2} + m_2 l_1 l_2\dot\theta_2^2 s_{1-2} + (m_1+m_2)g l_1 s_1 = \tau_1$$
$$m_2 l_2^2\ddot\theta_2 + m_2 l_1 l_2\ddot\theta_1 c_{1-2} - m_2 l_1 l_2\dot\theta_1^2 s_{1-2} + m_2 g l_2 s_2 = \tau_2$$

**행렬형 (p18)** — $M(\mathbf q)\ddot{\mathbf q} + C(\mathbf q,\dot{\mathbf q})\dot{\mathbf q} + \mathbf g(\mathbf q) = \boldsymbol\tau$
$$M(\boldsymbol\theta)=\begin{bmatrix}(m_1+m_2)l_1^2 & m_2 l_1 l_2 c_{1-2}\\[2pt] m_2 l_1 l_2 c_{1-2} & m_2 l_2^2\end{bmatrix},\quad
C(\boldsymbol\theta,\dot{\boldsymbol\theta})=\begin{bmatrix}0 & m_2 l_1 l_2\dot\theta_2 s_{1-2}\\[2pt] -m_2 l_1 l_2\dot\theta_1 s_{1-2} & 0\end{bmatrix}$$
$$\mathbf g(\boldsymbol\theta)= g\begin{bmatrix}(m_1+m_2)l_1 s_1\\ m_2 l_2 s_2\end{bmatrix}$$
슬라이드가 항을 **Mass / Coriolis·centrifugal / Gravity**로 색깔 분리해 보여줌 → 표준형 3항 구조를 눈으로 익히게 함.

**교수님 표현**: "$\tau_1$ 식엔 $m_1$과 $m_2$가 **둘 다** 걸리고($l_1$ 아래 전체), $\tau_2$엔 **$m_2$만**(2번 진자 이하). 가운데 $\dot\theta$ 곱 항이 **코리올리**, 미분 안 들어간 게 **그래비티**."

**⭐ 시험 대비**
- **재현 목표**: 2-link(이중진자) 운동방정식 유도 → $M,C,g$로 분해. 2R 예제(아래 §13)와 **쌍둥이**이므로 함께 연습.
- **자주 틀리는 지점**: $c_{1-2}=\cos(\theta_1-\theta_2)$의 시간미분에서 코리올리/원심 항이 나옴. $\frac{d}{dt}c_{1-2}=-s_{1-2}(\dot\theta_1-\dot\theta_2)$ 처리 실수 주의.

---

## 7. Center of Mass (COM) ★★

> 동역학은 **강체(rigid body)** 기반 → 시간이 지나도 형상·밀도분포가 안 변하므로 **각 링크의 COM을 기준**으로 해석한다.

**설명 흐름** (p22–24)
- COM 정의(p22): "**모든 질량이 집중**되어 있고, **모든 외력이 작용**한다고 볼 수 있는 점." (정지마찰·평형 분석의 기준)
- 입자계(set of particles, p23): $\displaystyle \mathbf p_c = \frac{\sum_i \mathbf p_i m_i}{\sum_i m_i}$
  - **예**: 정삼각형 꼭짓점 3개에 같은 질량 $m$ → $x_c=0$(대칭), $\displaystyle y_c=\frac{Hm+0+0}{3m}=\frac{H}{3}$. "**대칭(symmetry)을 활용하면 적분 없이** 바로 구한다"는 게 교수님 강조.
- 강체(rigid body, p24): $\displaystyle \mathbf p_c = \frac1m\int_m \tilde{\mathbf p}\,dm = \frac1m\int_V \tilde{\mathbf p}\,\rho\,dV$. 기준프레임이 COM이면 $\mathbf p_c=0$.

**COM의 속도·Jacobian** (p25)
- 링크 $i$의 COM 선속도/각속도: $\mathbf v_{ci}=J_v^i\dot{\mathbf q}$, $\boldsymbol\omega_i=J_\omega^i\dot{\mathbf q}$
- prismatic: $\begin{bmatrix}\mathbf z_{k-1}\\ \mathbf 0\end{bmatrix}$, revolute: $\begin{bmatrix}\mathbf z_{k-1}\times(\mathbf p_{ci}-\mathbf p_{k-1})\\ \mathbf z_{k-1}\end{bmatrix}$ — **엔드이펙터 대신 COM 기준**으로 잡은 점만 다르고 식 구조는 이전 강의의 geometric Jacobian과 동일.
- **교수님 표현**: "예전 자코비안을 **COM 기준으로 다시 푼 것뿐**. 왜냐면 자중·중력을 고려하려면 끝점이 아니라 COM에서 중력을 곱해야 하니까."

**⭐ 시험 대비**: 대칭 활용 COM 계산(계산형), 2R 예제에서 $J_v^i$를 COM 위치 미분으로 구하는 과정(§13)과 직결.

---

## 8. Inertia Tensor (관성텐서) ★★

**설명 흐름** (p28–30) — "회전에서 **질량의 역할**을 하는 게 관성텐서. 선운동의 $m$ ↔ 회전의 $\mathcal I$."

**대표 예 (p28)** — 일반물리/적분으로 유도했던 것들
- Solid sphere: $\mathcal I = \tfrac25 mr^2\,\mathbf I_3$
- Hollow sphere: $\mathcal I = \tfrac23 mr^2\,\mathbf I_3$
- **교수님 강조(자문자답)**: "같은 질량이면 solid vs hollow 중 누가 관성이 더 클까? **hollow**. 질량이 **축에서 먼 껍질**에 몰려 있으니까 $\tfrac23 > \tfrac25$." (대칭이라 $x,y,z$축 다 동일, 비대각=0)
- Cylinder, Box도 예시: 대칭이면 **메인 대각선 외 성분은 0**.

**Box 적분 예제 (p30)** — 밀도 $\rho=\dfrac{m}{V}=\dfrac{m}{abc}$ (균일)
$$I_{xx}=\int(y^2+z^2)\rho\,dV = \rho\!\int_{-c/2}^{c/2}\!\!\int_{-b/2}^{b/2}\!\!\int_{-a/2}^{a/2}\!(y^2+z^2)\,dx\,dy\,dz = \frac{m}{12}(b^2+c^2)$$
$$I_{yy}=\frac{m}{12}(a^2+c^2),\qquad I_{zz}=\frac{m}{12}(a^2+b^2)$$
- "$x$축 기준 회전이면 축으로부터 거리는 $y,z$ → $I_{xx}\propto y^2+z^2$." 대칭이라 cross term(곱관성) = 0.

**프레임 변환 (p29)** — 시험·예제에 직접 쓰임
$$\boxed{\,^0\mathcal I = {}^0R_B\,{}^B\mathcal I\,{}^0R_B^{\,T}\,}$$
- 각운동량 $^0\mathbf h = {}^0\mathcal I\,{}^0\boldsymbol\omega$에서 출발, $\{B\}$(링크프레임)에서 본 텐서를 $\{0\}$(베이스)로 옮긴 것.
- **이유**: 관성텐서는 **링크 프레임에서 써야 상수($\tilde{\mathcal I}$)**. 그래서 운동에너지 식에서 $R_i\tilde{\mathcal I}_i R_i^T$ 형태로 베이스 기준 텐서를 만든다.

**⭐ 시험 대비**: solid/hollow 비교(개념형), box $\frac{m}{12}(\cdot)$ 공식, **$^0\mathcal I = R\tilde{\mathcal I}R^T$** 변환. 평면 2R에선 $z$축 회전만이라 $I_{zz}$만 살아남음(§13 핵심).

---

## 9. 로봇의 Kinetic/Potential Energy & Mass Matrix $M(q)$ ★★★

**설명 흐름** (p31–33)

**Kinetic Energy** (p31–32)
- 링크 $i$ 하나: **선운동 + 회전운동** 두 항 (베이스 프레임 {0})
$$\mathcal T_i = \tfrac12 m_i \mathbf v_{ci}^T\mathbf v_{ci} + \tfrac12 \boldsymbol\omega_i^T\mathcal I_i\boldsymbol\omega_i$$
- 관성텐서를 **링크 프레임의 상수 $\tilde{\mathcal I}_i$**로 쓰면: $\tfrac12 \boldsymbol\omega_i^T R_i\tilde{\mathcal I}_i R_i^T\boldsymbol\omega_i$.
- $\mathbf v_{ci}=J_v^i\dot{\mathbf q}$, $\boldsymbol\omega_i=J_\omega^i\dot{\mathbf q}$ 대입 → joint 변수 표현:
$$\mathcal T_i = \tfrac12 m_i\big(\dot{\mathbf q}^T J_v^{iT}\big)\big(J_v^i\dot{\mathbf q}\big) + \tfrac12\big(\dot{\mathbf q}^T J_\omega^{iT}\big)R_i\tilde{\mathcal I}_i R_i^T\big(J_\omega^i\dot{\mathbf q}\big)$$
- **전체 로봇**: $1$~$n$ 링크 합산 → $\dot q$를 양옆으로 빼면 가운데가 행렬이 됨:
$$\mathcal T = \sum_{i=1}^n\mathcal T_i = \tfrac12\dot{\mathbf q}^T\!\underbrace{\left(\sum_{i=1}^n m_i J_v^{iT}J_v^i + J_\omega^{iT}R_i\tilde{\mathcal I}_i R_i^T J_\omega^i\right)}_{\textstyle M(\mathbf q)}\!\dot{\mathbf q}$$
$$\boxed{\,M(\mathbf q) = \sum_{i=1}^n \big(m_i J_v^{iT}J_v^i + J_\omega^{iT}R_i\tilde{\mathcal I}_i R_i^T J_\omega^i\big)\,},\qquad \mathcal T(\mathbf q,\dot{\mathbf q}) = \tfrac12\dot{\mathbf q}^T M(\mathbf q)\dot{\mathbf q}$$
- **$M(q)$의 성질(★ 시험)**: **symmetric**(대칭), **positive definite**(양정부호).
- **교수님 표현**: "운동에너지에서 **각속도($\dot q$)에 걸리는 부분만 빼고** 가운데 관성 덩어리를 통째로 묶은 게 **mass matrix**. 그래서 inertia matrix라고도 부른다."

**Potential Energy** (p33) — 훨씬 간단
$$U_i = -m_i\,\mathbf g_0^T\,\mathbf p_{ci},\qquad U(\mathbf q) = -\sum_{i=1}^n m_i\,\mathbf g_0^T\,\mathbf p_{ci}$$
- $\mathbf g_0$: 중력가속도 벡터, $\mathbf p_{ci}$: 링크 $i$ COM 위치(=$\mathbf p_{ci}(\mathbf q)$, joint에 의존).
- **note(슬라이드)**: 모든 것은 **base frame** 기준으로 표현 (운동에너지와 호환되도록; 관성텐서만 링크프레임).

**⭐ 시험 대비**
- **재현 목표**: $\mathcal T=\tfrac12\dot q^T M\dot q$, $U=-\sum m_i\mathbf g_0^T\mathbf p_{ci}$를 세우고, $M(q)=\sum(m_i J_v^{iT}J_v^i + J_\omega^{iT}R\tilde{\mathcal I}R^T J_\omega^i)$를 쓸 수 있어야.
- **핵심 구분 암기**: $M$의 두 덩어리 = **선속도 Jacobian항 + 회전(관성텐서)항**. $M$은 **대칭·양정부호**.

---

## 10. Robot Dynamics via Euler–Lagrange — 항별 유도 + Christoffel ★★★

> 이번 강의의 **메인 새 내용**. $\mathcal L=T-U$를 E-L 방정식에 넣어 표준형이 나오는 과정을 **항별로 직접 전개**. 교수님이 "텀바이텀으로 보자"며 칠판식으로 진행 → 시험 유도 1순위.

**설명 흐름** (p35–43)

**(1) 표준형 도출** (p35–37)
- $\mathcal L = \tfrac12\dot{\mathbf q}^T M(\mathbf q)\dot{\mathbf q} + \sum_i m_i\mathbf g_0^T\mathbf p_{ci}$ (즉 $T-U$)
- E-L 벡터형 대입 후 정리:
$$M(\mathbf q)\ddot{\mathbf q} + \underbrace{\Big(\dot M(\mathbf q) - \tfrac12\dot{\mathbf q}^T\big(\tfrac{\partial M(\mathbf q)}{\partial \mathbf q}\big)^{\!T}\Big)}_{C(\mathbf q,\dot{\mathbf q})}\dot{\mathbf q} + \underbrace{\Big(-\sum_i J_v^{iT}(\mathbf q)\,m_i\mathbf g_0\Big)}_{\mathbf g(\mathbf q)} = \boldsymbol\tau_{tot}$$
- → 표준형 $M\ddot q + C\dot q + g = \tau_{tot}$. 슬라이드 note: $J_v^i(\mathbf q)=\dfrac{\partial \mathbf p_{ci}}{\partial \mathbf q}$.

**(2) 주요 성질** (p37)
- $M(\mathbf q)=M^T(\mathbf q)$ (대칭), $\quad M(\mathbf q)\succ 0$ (양정부호)
- $\dot M(\mathbf q) - 2C(\mathbf q,\dot{\mathbf q})$ 는 **skew-symmetric**(반대칭). ★ 제어이론에서 자주 쓰는 성질.

**(3) 항별 유도 + Christoffel 기호** (p40) — 시험에 그대로 나올 수 있는 형태
$$\boxed{\,\sum_{j=1}^n m_{ij}(\mathbf q)\ddot q_j + \sum_{j=1}^n\sum_{k=1}^n c_{ijk}\,\dot q_k\dot q_j - \sum_{k=1}^n (J_v^k)^T m_k\mathbf g_0 = \tau_i\,},\quad i=1,\dots,n$$
**First-kind Christoffel symbols**:
$$\boxed{\,c_{ijk} = \frac12\left(\frac{\partial m_{ij}}{\partial q_k} + \frac{\partial m_{ik}}{\partial q_j} - \frac{\partial m_{jk}}{\partial q_i}\right)\,},\qquad c_{ijk}=c_{ikj}$$
- **유도 트릭(교수님 강조)**: 속도곱 항을 정리할 때 "$a$ 대신 $k$를 써도 같은 표기"라는 점을 이용, $\tfrac12(m_{ij,k}+m_{ik,j}-m_{jk,i})$로 묶어 **Christoffel 형태**로 만든다. ($c_{ijk}=c_{ikj}$ 대칭 → 연산 절반 생략 가능.)

**(4) 각 항의 해석** (p41–42)
- **관성항** $m_{ij}$:
  - $m_{ii}$(대각): 다른 관절 정지 시 **관절 $i$ 자신의 관성**.
  - $m_{ij}$($i\ne j$, 비대각): 관절 $j$의 가속이 관절 $i$에 주는 **coupling(연성) 효과**.
- **속도항**:
  - $c_{ijj}\dot q_j^2$: 관절 $j$ 속도에 의한 **centrifugal(원심력)**, 관절 $i$에 작용.
  - $c_{ijk}\dot q_j\dot q_k$ ($j\ne k$): 관절 $j,k$ 속도에 의한 **Coriolis** 효과.
- **중력항**: $g_i(\mathbf q) = -\sum_k (J_v^k)^T m_k\mathbf g_0$ — 자중에 의해 관절 $i$에 필요한 토크.

**⭐ 시험 대비**
- **재현 목표**: $\mathcal L=T-U$ → E-L → **항별식 + Christoffel 정의** 유도. $c_{ijk}$ 공식 암기·적용.
- **예상 유형**: 유도형(표준형 도출), 계산형(주어진 $M$에서 $c_{ijk}$ 계산).
- **자주 틀리는 지점**: $c_{ijk}$ 세 편미분의 **부호와 인덱스 순서**($\partial m_{ij}/\partial q_k + \partial m_{ik}/\partial q_j - \partial m_{jk}/\partial q_i$). $c_{ijk}=c_{ikj}$ 대칭.
- **핵심 구분 암기**: $\dot q_j^2$=원심력 / $\dot q_j\dot q_k$=코리올리 / 대각 $m_{ii}$=자기관성 / 비대각=coupling.

---

## 11. Generalized Force 구성요소 ★★

**설명 흐름** (p38) — 표준형 우변 $\boldsymbol\tau_{tot}$를 세부 분해.
$$\boxed{\,\boldsymbol\tau_{tot} = \boldsymbol\tau + \boldsymbol\tau_{ext} - \boldsymbol\tau_m - \boldsymbol\tau_{fr}\,}$$
- $\boldsymbol\tau$: **모터가 관절에 가하는 토크**(입력 토크).
- $\boldsymbol\tau_{ext} = J^T\begin{bmatrix}\mathbf f_{ext}\\ \mathbf m_{ext}\end{bmatrix}$: 외력·외부모멘트. ($J$=힘 작용점의 geometric Jacobian.)
- $\boldsymbol\tau_m = \mathcal I_m\ddot{\mathbf q}$: **모터 자체의 관성**에 의한 토크($\mathcal I_m$=motor inertia tensor).
- $\boldsymbol\tau_{fr} = F_s\,\mathrm{sgn}(\dot{\mathbf q}) + F_v\dot{\mathbf q}$: 관절 마찰. $F_s$=정지(Coulomb) 대각마찰행렬, $F_v$=점성(viscous) 대각마찰행렬. ($\mathrm{sgn}$: $\dot q>0\to1,\,=0\to0,\,<0\to-1$, 마찰 방향 결정용.)

**교수님 표현**: "모터관성·마찰은 표준문제에선 **종종 무시**하지만, **정밀 제어**하려면 뒤 두 항까지 다 고려해야 한다. 외력은 문제 예제풀이에서도 다뤘다."

**⭐ 시험 대비**: 네 항의 의미·부호 구분(개념형). $\mathrm{sgn}$이 마찰 **방향**을 주는 이유(velocity 부호 기반)를 설명할 수 있게.

---

## 12. Procedure for the Calculation — 5단계 ★★★

**설명 흐름** (p43) — "실제 계산은 이 5스텝으로 정리된다." 2R 예제(§13)가 이 절차의 정확한 시연.

$$
\begin{aligned}
&\textbf{Step 1.}\ \text{관성행렬 } M(\mathbf q)=\sum_i\big(m_i J_v^{iT}J_v^i + J_\omega^{iT}R_i\tilde{\mathcal I}_i R_i^T J_\omega^i\big)\ \ (\text{CoM Jacobian 필요})\\
&\textbf{Step 2.}\ \text{Coriolis 행렬 } C,\ \ c_{ij}=\sum_{k}c_{ijk}\dot q_k,\quad c_{ijk}=\tfrac12\big(\tfrac{\partial m_{ij}}{\partial q_k}+\tfrac{\partial m_{ik}}{\partial q_j}-\tfrac{\partial m_{jk}}{\partial q_i}\big)\\
&\textbf{Step 3.}\ \text{중력벡터 } g_i=-\sum_{k}(J_{v}^{k})^T m_k\mathbf g_0\\
&\textbf{Step 4.}\ \text{관절별 generalized force } \boldsymbol\tau_{tot}\ \text{결정}\\
&\textbf{Step 5.}\ \text{동역학 모델 } M(\mathbf q)\ddot{\mathbf q}+C(\mathbf q,\dot{\mathbf q})\dot{\mathbf q}+\mathbf g(\mathbf q)=\boldsymbol\tau_{tot}\ \text{조립}
\end{aligned}
$$
- Step 2,3 순서는 무관. "결국 가속도항·속도항·중력항 + 좌우 일반화 힘."

**⭐ 시험 대비**: **이 5단계 = 동역학 계산 문제의 만능 템플릿.** 어떤 매니퓰레이터가 나와도 이 순서로 풀면 된다. 순서·각 step 산출물($M,C,g,\tau$)을 외울 것.

---

## 13. Example 1 — 2R(RR) 평면 매니퓰레이터 완전 풀이 ★★★

> **시험 직결 1순위.** 교수님: "기말고사 때 도움이 될 것", "두 번째 예제도 캠퍼스에 올린다." 5단계 절차를 그대로 시연.

**문제 설정** (p45)
- 2-DOF RR 평면 매니퓰레이터. 질량 $m_1,m_2$. 링크1의 $z$축(COM 기준) 관성 $I_{zz1}$, 링크2 $I_{zz2}$. 길이 $l_1$, COM까지 거리 $l_{c1},l_{c2}$. 평면이라 회전은 $z$축뿐 → **관성텐서는 $I_{zz}$만 살아남음.**
- 표기: $c_1=\cos q_1$, $s_{12}=\sin(q_1+q_2)$, $c_{12}=\cos(q_1+q_2)$ 등.

### Step 1 — Inertia Matrix (p46–49)
**COM 위치**: $x_1=l_{c1}c_1,\ y_1=l_{c1}s_1$; $\quad x_2=l_1 c_1 + l_{c2}c_{12},\ y_2=l_1 s_1 + l_{c2}s_{12}$
**선속도 Jacobian** (COM 위치를 $q$로 편미분):
$$J_v^1=\begin{bmatrix}-l_{c1}s_1 & 0\\ l_{c1}c_1 & 0\\ 0 & 0\end{bmatrix},\quad
J_v^2=\begin{bmatrix}-l_1 s_1 - l_{c2}s_{12} & -l_{c2}s_{12}\\ l_1 c_1 + l_{c2}c_{12} & l_{c2}c_{12}\\ 0 & 0\end{bmatrix}$$
**각속도 Jacobian** ($z$축 회전만): $J_\omega^1=[\mathbf z_0\ \mathbf 0]=\begin{bmatrix}0&0\\0&0\\1&0\end{bmatrix}$, $\quad J_\omega^2=[\mathbf z_0\ \mathbf z_1]=\begin{bmatrix}0&0\\0&0\\1&1\end{bmatrix}$
**곱 계산** (p47–48):
$$J_v^{1T}J_v^1=\begin{bmatrix}l_{c1}^2 & 0\\ 0 & 0\end{bmatrix},\quad
J_v^{2T}J_v^2=\begin{bmatrix}l_1^2+2l_1 l_{c2}c_2+l_{c2}^2 & l_1 l_{c2}c_2+l_{c2}^2\\ l_1 l_{c2}c_2+l_{c2}^2 & l_{c2}^2\end{bmatrix}$$
$$(J_\omega^{1T}R_1)\tilde{\mathcal I}_1(R_1^T J_\omega^1)=\begin{bmatrix}I_{zz1} & 0\\ 0 & 0\end{bmatrix},\quad
(J_\omega^{2T}R_2)\tilde{\mathcal I}_2(R_2^T J_\omega^2)=\begin{bmatrix}I_{zz2} & I_{zz2}\\ I_{zz2} & I_{zz2}\end{bmatrix}$$
- "$z$축으로만 도니까 나머지 텐서 항은 **다 날아가고** $I_{zz1},I_{zz2}$만 남는다."
**합산 → $M(q)$** (p49, **대칭**):
$$M(\mathbf q)=\begin{bmatrix}
m_1 l_{c1}^2 + m_2\!\left(l_1^2+2l_1 l_{c2}c_2+l_{c2}^2\right)+I_{zz1}+I_{zz2} & m_2\!\left(l_1 l_{c2}c_2+l_{c2}^2\right)+I_{zz2}\\[4pt]
m_2\!\left(l_1 l_{c2}c_2+l_{c2}^2\right)+I_{zz2} & m_2 l_{c2}^2 + I_{zz2}
\end{bmatrix}$$
즉 $m_{11},m_{12}=m_{21},m_{22}$:
- $m_{11}=m_1 l_{c1}^2 + m_2(l_1^2+2l_1 l_{c2}c_2+l_{c2}^2)+I_{zz1}+I_{zz2}$
- $m_{12}=m_{21}=m_2(l_1 l_{c2}c_2+l_{c2}^2)+I_{zz2}$
- $m_{22}=m_2 l_{c2}^2+I_{zz2}$

### Step 2 — Coriolis/Centrifugal Matrix (p50–52)
$c_{ijk}=\tfrac12(\partial_k m_{ij}+\partial_j m_{ik}-\partial_i m_{jk})$, $c_{ij}=\sum_{k=1}^2 c_{ijk}\dot q_k$. $c_{ijk}=c_{ikj}$ → 계산할 것은 $c_{111},c_{112},c_{122},c_{211},c_{212},c_{222}$.
$$c_{111}=\tfrac12\partial_{q_1}m_{11}=0,\qquad c_{222}=\tfrac12\partial_{q_2}m_{22}=0$$
$$c_{112}=c_{121}=\tfrac12\partial_{q_2}m_{11}=-m_2 l_1 l_{c2}s_2 \equiv h,\qquad c_{122}=-m_2 l_1 l_{c2}s_2 = h$$
$$c_{211}=-\tfrac12\partial_{q_2}m_{11}=m_2 l_1 l_{c2}s_2 = -h,\qquad c_{212}=c_{221}=0$$
**여기서 $\boxed{h \equiv -m_2 l_1 l_{c2}s_2}$.** 대입(p52):
$$c_{11}=h\dot q_2,\quad c_{12}=h(\dot q_1+\dot q_2),\quad c_{21}=-h\dot q_1,\quad c_{22}=0$$
$$\boxed{\,C(\mathbf q,\dot{\mathbf q})=\begin{bmatrix}h\dot q_2 & h(\dot q_1+\dot q_2)\\ -h\dot q_1 & 0\end{bmatrix}\,}$$
- 슬라이드 note: "**$C$ 행렬은 유일하지 않다**(other options 존재)." — $\dot M-2C$ 반대칭만 지키면 됨.

### Step 3 — Gravity Vector (p53–54)
$g_i=-\sum_{k=1}^2 (J_{v_i}^k)^T m_k\mathbf g_0$, $\mathbf g_0=[0,\,-g,\,0]^T$.
$$g_1 = g\big((m_1 l_{c1}+m_2 l_1)c_1 + m_2 l_{c2}c_{12}\big),\qquad g_2 = g\,m_2 l_{c2}c_{12}$$

### Step 5 — Dynamic Model (p55)
$$\begin{bmatrix}m_{11}&m_{12}\\ m_{21}&m_{22}\end{bmatrix}\!\begin{bmatrix}\ddot q_1\\ \ddot q_2\end{bmatrix}
+\begin{bmatrix}h\dot q_2 & h(\dot q_1+\dot q_2)\\ -h\dot q_1 & 0\end{bmatrix}\!\begin{bmatrix}\dot q_1\\ \dot q_2\end{bmatrix}
+\begin{bmatrix}g\big((m_1 l_{c1}+m_2 l_1)c_1 + m_2 l_{c2}c_{12}\big)\\ g\,m_2 l_{c2}c_{12}\end{bmatrix}
=\begin{bmatrix}\tau_1\\ \tau_2\end{bmatrix}$$
- 외력 없음 → 우변은 액추에이터 토크 $\tau_1,\tau_2$만. (외력 있으면 $\tau_{ext}$ 추가.)
- 값(길이·질량)이 주어지면 숫자 대입으로 실제 행렬 산출 → **제어/시뮬레이션에 사용.**

**⭐ 시험 대비**
- **재현 목표**: 2R 매니퓰레이터를 **5단계 그대로** 손으로 완주($M$→$C$→$g$→조립). $h=-m_2 l_1 l_{c2}s_2$ 정의와 $C$행렬 구조 암기.
- **예상 유형**: **계산형 + 유도형** (이번 강의 최대 출제 후보). 이중진자(§6)와 형상 동일 → 교차 연습.
- **자주 틀리는 지점**: ① $J_v^i$를 **COM 위치**($l_{c}$) 미분으로(엔드이펙터 길이 $l$ 아님 — 단 $x_2$엔 $l_1$이 들어감에 주의), ② 평면이라 $I_{zz}$만 남김, ③ $m_{11}$의 $2l_1 l_{c2}c_2$ 교차항, ④ $c_{ijk}$ 부호.

---

## 곁가지·여담 모음 (★)

- **2학년 동역학/일반물리 선수지식 환기**: "Euler–Lagrange, Newton–Euler는 2학년 동역학에서 배웠다", "Craig 책에 코리올리 예제 있다", "관성텐서 적분은 일반물리/구조에서 했다", "이중진자는 어드밴스드 공학수학에서 물어본 적 있다." → **시험에서 당연히 안다고 가정하는 부분.**
- **재료 이야기**: "링크는 대부분 금속(알루미늄은 가볍지만), 큰 힘 견디려고 stiffness·density 높은 금속 쓰면 **자중이 상당** → 자중 버틸 토크도 무시 못 함." (중력항이 왜 중요한지)
- **마지막 시간**: "시험 전 마지막 시간이라 더 진도 안 나가고, **연구실에서 어떤 연구 하는지 소개**하겠다"로 강의 마무리. (연구 소개는 전사 끝부분, 시험 무관)

---

## 마지막 점검

**슬라이드 vs 전사 / ASR 추정 교정**
- [전사 추정] "비텀/위턴 오일러" → **Newton–Euler**, "크리스토 토벨/크리스토파일" → **Christoffel**, "스키 시메트릭" → **skew-symmetric**, "포지티브 디퓨넷" → **positive definite**, "인열차/이널샤/인허시아 텐서" → **inertia tensor**, "제너럴라이즈드 폴스/홀스" → **generalized force**, "투알/2알 로봇" → **2R robot**, "디스코스" → **viscous**. (모두 슬라이드 영어 용어를 ground truth로 교정.)
- 이번 전사본엔 RL 가이드의 "필드/휠 로봇" 혼동 사례는 **없음**.

**그림 의존으로 복원한 부분**
- p13–15 펜듈럼 좌표(원점=pivot, $x$우/$y$상), p16 이중진자 각 정의($\theta_1,\theta_2$ 연직 기준), p45 2R 로봇의 $l_1,l_{c1},l_{c2},q_1,q_2$ 배치 — 모두 PNG 렌더링으로 직접 확인해 복원함.

**슬라이드 표기 보정 (둘 다 표시)**
- p16 Double pendulum **Answer**의 압축 표기는 작게 렌더되어 일부 첨자가 흐렸으나, **p17 고해상 항별 유도**와 일치하도록 $l_1^2,\ l_1 l_2,\ l_2^2$ 차원으로 정정해 기재함. p18 행렬형($M,C,g$)도 동일 기준.
- p36의 $C(q,\dot q)=\dot M - \tfrac12\dot q^T(\partial M/\partial q)^T$ 형태는 슬라이드 표기 그대로 옮김(편미분 텐서 표기는 강의 관례).

**강의 경계(불확실 — 확인 필요)**
- 전사본 앞부분(타임스탬프 없는 단락)은 Introduction~에너지~표준형까지, 뒷부분(`참석자 1 00:00~`)은 **에너지 복습 → 항별 유도 → 2R 예제**를 다룸. 교수님이 "**지난 시간에** 키네틱/포텐셜 에너지를 유도했다"고 명시 → **§7–9(COM·관성텐서·에너지)는 지난 강의에서 유도, 이번에 복습**했을 가능성이 큼. 본 정리본은 **이 PDF 덱 전체**를 한 묶음으로 정리했으니, 06/11 당일 신규 진행분은 **§10(항별 유도)·§11–12·§13(2R 예제)** 중심으로 보면 됨.

**교수님이 남긴 예고·숙제**
- **두 번째 예제 풀이를 캠퍼스에 업로드** → 반드시 확인(기말 도움). 
- Exercise: **Double pendulum 직접 풀어 확인**(§6).
- Newton–Euler 방식은 이번 강의에서 **개념만 언급**(실시간 inverse dynamics용)하고 유도는 하지 않음 → 출제 시 E-L과의 **용도 비교** 수준일 가능성.

