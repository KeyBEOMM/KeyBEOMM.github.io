---
title: "19강 Imitaition Learning"
description: "Imitation Learning"
author: KEYBEOMM
date: 2026-05-31 17:00:00 +0900
categories: [RL]
tags: [RL, Imitation Learning]
math: true
---

# 19강 Imitation Learning-1 (Behavioral Cloning) 강의 정리본

---

## 0. 강의 한눈에 보기

**교수님의 프레이밍:**
강의 시작에 "한 학기 과목 통틀어서 오늘이 제일 쉬울 것"이라고 예고. 강화학습이 아닌 Supervised Learning 방식이기 때문. 이전 강의들(Model-free RL, Model-based RL)에서 힘들었던 학생들에게 숨 돌릴 수 있는 강의로 제시.

**강의의 위치:**
- 지난 강의: Model-based RL — 경험으로 모델을 예측 → 플래닝 or 시뮬레이션 에피소드 생성
- 이번 강의: **Imitation Learning** — 또 다른 방식의 폴리시 학습
- 다음 강의: Imitation Learning 2회차 (Inverse RL, GAIL 등)

**다룬 개념 목록 및 중요도:**

| 개념 | 중요도 |
|------|--------|
| Imitation Learning 개요 및 분류 | ★ |
| Notation | ★ |
| Behavioral Cloning (BC) 핵심 개념 | ★★★ |
| BC의 핵심 문제: Distribution Shift / Covariate Shift | ★★★ |
| ALVINN (1988) | ★★ |
| DAgger | ★★★ |
| DART | ★★ |
| BC Summary | ★★ |
| LLM과 Imitation Learning (SFT, RLHF) | ★★ |

---

## 1. 강의 배경: Imitation Learning이란 무엇인가 ★

**교수님의 설명 흐름:**
지금까지 배운 방식들을 복습하며 IL을 위치시킨다. Model-free RL은 직접 경험한 리워드로 학습, Model-based RL은 모델을 예측해서 추가 에피소드를 만들어내는 방식. 이제 세 번째 방법인 Imitation Learning을 소개한다.

**핵심 정의 (강의자료 기준):**
- Goal: 전문가의 데모스트레이션을 모방하는 폴리시 학습
- 리워드를 최대화하는 것이 아니라 **행동을 모방**하는 것 (Imitation Learning ≠ RL)
- **Learning from Demonstration (LfD)**

**IL의 세 가지 종류 (강의자료 기준):**

| 방법 | 핵심 | 특징 |
|------|------|------|
| **Behavioral Cloning (BC)** | 전문가 행동 직접 복제 → SL로 환원 | 리워드 불필요, 사전 수집 데모 사용 |
| **Inverse RL (IRL)** | 데모에서 리워드 함수를 역으로 학습 | 리워드 학습 후 RL 수행 |
| **GAIL** (Generative Adversarial Imitation Learning) | GAN 방식으로 폴리시 학습 | Generator-Adversarial-Discriminator 구조 |

**교수님의 말:** "오늘 배우고 나서 다시 한 번 살펴보도록 하겠습니다" — IL 전체 분류를 큰 그림으로만 제시하고 오늘은 BC만 다룬다고 명시.

---

## 2. Notation ★

**핵심 정의 (강의자료 기준):**
- State: $s$ (input)
- Action: $a$ (output)
- Policy: $\pi_\theta$
  - 결정론적: $\pi_\theta(s) \to a$
  - 확률적: $\pi_\theta(s) \to P(a)$
- State transition probability (dynamics): $P(s'|s, a)$
  - **폴리시는 모름** — 환경/시뮬레이터에 해당
- Expert policy: $\pi^*$
- Demonstrations: $D = \{(s_i, a_i)\}_{i=1}^N$

**교수님의 설명 방식:**
State = "인풋", Action = "아웃풋"으로 바로 연결. "Demonstrations는 스테이트-액션 쌍들의 시퀀스"로 직관적으로 표현.

---

## 3. Behavioral Cloning (BC) ★★★

**— ★★★ 근거: "결국에는 슈퍼바이즈드 러닝이에요"라고 명시적으로 두 번 이상 반복. 이 개념이 이번 강의 전체의 핵심이며, 이후 SFT 연결까지 이어지는 축.**

**교수님의 설명 흐름:**
"결국에는 슈퍼바이즈드 러닝이에요"라고 바로 핵심을 꺼낸다. 전문가가 특정 스테이트에서 특정 액션을 한 기록이 있으면, 그게 정답 쌍이다. 스테이트를 입력으로 넣어서 그 액션이 나오는 모델을 학습하는 것. 리워드가 없는 게 아니라, **정답 액션과의 차이**가 로스 역할을 한다.

**핵심 정의 / 수식 (강의자료 기준):**

$$\arg\min_\theta E_{(s,a^*) \sim P^*} L\left(\mathbf{a}^*, \pi_\theta(\mathbf{s})\right)$$

- $P^* = P(s|\pi^*)$: 전문가가 방문한 스테이트들의 분포
- **1-step deviation error를 최소화**하는 것으로 해석 가능
- Idea: Treat imitation learning as a supervised learning problem!

**교수님이 쓴 비유·예시·표현:**

*자율주행 비유 (매우 상세하게 설명):*
> "집에 막 그런 핸들 이런 거 엄청 비싼 거 있지 않습니까? 거기에 앉혀놓고 게임을 시켜요. 운전을 하면 처음에 기록을 하는 거죠. 스테이트 원에서 어떤 액션을 했고... 전문가가 어떤 숙련된 운전 기술로 이렇게 운전해서 결승점에 도달을 하면 우리는 기록이 남잖아요. 그걸로 어떤 스테이트에서 어떤 액션이 나와야 되는지를 학습합니다."

*폴리시 = ChatGPT 비유:*
> "우리가 LLM에 '내가 현재 처한 상황이 이래, 빚이 얼마고 대출은 얼마고... 오늘 뭘 해야 될까 1번 일한다, 2번 앉아서 고민한다, 3번 다 잊고 놀러 간다'라고 했을 때 챗GPT가 3번을 선택해줬잖아요. 그때 챗GPT는 폴리시입니다."

*리워드의 역할:*
> "오른쪽으로 가야 하는데 내가 왼쪽으로 갔다, 그러면 그 차이만큼 로스가 발생한다. 리워드는 없어지면 안 됩니다."

*자율주행에 IL이 잘 맞는 이유:*
> "자율주행이라고 하는 것이 이미테이션 러닝이 가능한 이유 중에 하나가 액션 스페이스가 작아요. 스테이트는 물론 많을 수 있기는 한데 그것도 세그멘테이션을 통해서 길과 장애물만 간략하게 하게 된다면 사실 스테이트도 그렇게 복잡하지는 않습니다."

**주의·함정:**
- BC는 리워드가 필요 없는 것처럼 보이지만, 학습 과정에서 "정답 액션과의 차이"가 로스 역할을 함 → BC가 RL과 완전히 다른 차원의 것이라고 오해하지 말 것
- 교수님 직접 언급: "쉬운 거는 보통 성능이 안 좋습니다"

---

## 4. BC의 핵심 문제: Distribution Shift / Covariate Shift ★★★

**— ★★★ 근거: 전사본에서 가장 많은 시간을 할애한 개념. 여러 번 다른 표현으로 반복. 슬라이드에도 두 장에 걸쳐 설명. 이후 DAgger, SFT/RLHF 등장의 동기가 모두 이 문제에서 출발.**

**교수님의 설명 흐름:**
BC가 왜 쉽지만 성능이 안 좋은지를 IID 가정 위반으로 설명한다. Supervised Learning은 학습 데이터와 테스트 데이터가 같은 분포에서 나온다고 가정(IID)하는데, BC에서는 이것이 근본적으로 깨진다.

**IID 가정의 두 가지 전제 (강의자료 기준):**

| 조건 | 의미 in BC | 문제 |
|------|-----------|------|
| **Independent** | 각 샘플 $(s, a^*)$가 서로 독립 | 시퀀셜하므로 이전 스테이트가 다음 스테이트를 결정 → 독립 아님 |
| **Identically Distributed** | 모든 샘플이 같은 분포에서 추출 | 전문가 학습 분포 vs 실제 롤아웃 분포가 다름 |

**핵심 정의: Covariate Shift (강의자료 기준):**

$$P_{\text{train}}(s) \neq P_{\text{test}}(s), \quad P(a|s) \text{ is unchanged}$$

- Covariate = 머신러닝에서 "인풋" (통계학 용어) = RL에서는 "스테이트"
- 폴리시 자체는 같은데 들어오는 스테이트 분포가 달라짐
- 교수님 설명: "코베리에이트 시프트라고 하는 거는 내가 학습한 스테이트와 들어오는 스테이트가 달라요."

**교수님이 쓴 비유·예시·표현:**

> "전문가는 항상 자동차 게임을 해도 항상 가운데만 보고 항상 정도를 벗어나지 않아. 그렇기 때문에 전문가 시연으로부터 스테이트가 벗어나면 거기에 대한 이체는 학습할 기회가 없어요."

> "처음에는 잘 따라가다가 조금 벗어나게 되면, 액션이 달라질 거고, 액션이 달라지게 되면 스테이트의 차이가 점점점점 벌어지게 됩니다. 그러면 점점 어떤 액션이 나올지 가늠이 안 되겠죠."

> "나갈 수는 있지, 나갔는데 나 다시 돌아온다 상관없잖아요. 근데 그렇게 될 장치가 없다는 겁니다."

> "한 번 멀어지기 시작하면 거기에서 올바른 액션을 할 수 없다는 거지."

**해결책 (빠르게 언급, ★):**
- Importance Sampling
- Data augmentation
- Domain adaptation

→ 이 해결책들의 구체적 구현이 DAgger, DART로 이어짐.

---

## 5. ALVINN (1988) ★★

**교수님의 설명 흐름:**
"역사적으로 좀 보겠습니다. 너무 옛날 거 치고는 인풋의 퀄리티가 좋아가지고 좀 놀랐습니다"라며 BC의 최초 성공 사례로 소개.

**핵심 정보 (강의자료 기준):**
- **ALVINN**: Autonomous Land Vehicle In a Neural Network
- Dean A. Pomerleau, CMU, **NeurIPS 1988**
- Input: 30×32 이미지 + 8×32 LRF(라이다) 합쳐서 입력
- Single hidden layer (29 hidden units)
- **45개 direction output** → 핸들 방향 결정
- Data augmentation: image rotation, shift 사용

**교수님의 부연:**
> "88년 당시에는 뉴럴넷이라고 하는 것이 조금 이제 대안이 없어서 흥하던 시절입니다. 서포트 벡터 머신이 나오면서 거의 망하다가 요즘에는 다시 흥하고 있죠."

> "그 당시에 이미 8채널짜리 라이더를 이미 썼습니다. 보시면 아시겠지만 이미 세그멘테이션이 된 바닥을 가고 있습니다. 길과 길이 아닌가 확연히 구분됩니다."

**강조 포인트:** ★★ — 역사적 사례로 빠르게 소개. "의외로 퀄리티가 좋아서 놀랐다"는 감탄 표현 포함.

---

## 6. DAgger (Dataset Aggregation) ★★★

**— ★★★ 근거: "네 줄이 다입니다"라고 두 번 강조. Dyna-Q와 비교하며 "아직까지 이 개념 자체가 많이 사용된다"고 명시. 레이블링 bottleneck 계산($5분 \times 30FPS = 9000장$)까지 직접 해주면서 현실적 중요성 강조. 이후 RLHF까지 이어지는 핵심 연결 고리.**

**교수님의 설명 흐름:**
"그다음에 DAgger인데요. 이것도 되게 조금 참신한 아이디어라고 저는 생각이 듭니다." 먼저 지난 강의 Dyna-Q와 비교한다: "Dyna-Q가 엄청 간단한 개념이잖아요 — 알고리즘이 네 줄인가 정도밖에 안 되잖아요. 근데 아직까지도 그 개념 자체가 많이 사용이 되고 있습니다. 이 DAgger라고 하는 것도 네 줄입니다. 네 줄이 다인데."

**핵심 정보 (강의자료 기준):**
- Stéphane Ross, Geoffrey J. Gordon, J. Andrew Bagnell (CMU), **PMLR, 2011**
- 해결 전략: training 분포와 evaluation 분포를 같게 만들기
  - $\pi_\theta(a_t|o_t)$를 이용해 rollout → 그 상태에서 전문가 레이블 받음 → 데이터 합산

**알고리즘 (강의자료 기준):**
$$\text{1. train } \pi_\theta(\mathbf{a}_t|\mathbf{o}_t) \text{ from human data } \mathcal{D} = \{\mathbf{o}_1, \mathbf{a}_1, \ldots, \mathbf{o}_N, \mathbf{a}_N\}$$
$$\text{2. run } \pi_\theta(\mathbf{a}_t|\mathbf{o}_t) \text{ to get dataset } \mathcal{D}_\pi = \{\mathbf{o}_1, \ldots, \mathbf{o}_M\}$$
$$\text{3. Ask human to label } \mathcal{D}_\pi \text{ with actions } \mathbf{a}_t$$
$$\text{4. Aggregate: } \mathcal{D} \leftarrow \mathcal{D} \cup \mathcal{D}_\pi$$

**교수님이 쓴 비유·예시·표현:**

*전문가 시간 활용 비유 (전사본에서 가장 구체적으로 설명):*
> "비싼 전문가를 내가 10시간 쓸 수 있다고 하면, BC는 10시간 동안 운전을 해 주세요라고 해서 데이터를 수집해서 학습하는 거예요. DAgger는 일단 1시간만 운전해 주세요. 그럼 1시간 데이터로 모델을 학습합니다. 그걸로 실제로 게임을 했는데 이상한 길로 가요. 그러면 얘가 이상한 길로 가는데, 이 길에 대해서 어떻게 운전하면 되는지 정답을 알려주세요. 화살표를 그려주세요. 핸들을 그 상황에서 어떻게 했는지 답을 알려주세요."

> "그러니까 전문가의 시연 시간은 똑같다는 거예요. 근데 그 단계가 한 번에 다 데모스트레이션을 만든 게 아니라 한 번 만들면 그걸로 하고, 다 달아주세요 → 학습 → 해보고 → 또 다 달아주세요를 반복하는 방식."

**레이블링 Bottleneck (교수님이 계산까지 직접):*
> "1초에 30장이니까 5분이면 5 곱하기 60 곱하기 30 = 9천 장. 9천 개에 대해서 레이블링을 해달라고 하면 쉬운 일이 아니죠. 실제로는 그렇게 하기가 어려워요. 그랬기 때문에 차라리 내가 운전하는 게 낫지, 이 피지 보고 내가 다 달아주는 건 스트레스 아니겠습니까?"

**Practical Remedies (강의자료 기준):**

| 방법 | 내용 |
|------|------|
| **Automated Expert** | 시뮬레이터/룰 기반 레이블링 (예: "차선 가운데가 항상 정답") |
| **SafeDAgger / EnsembleDAgger** | 불확실한 스테이트만 쿼리 |
| **Temporal downsampling** | 30FPS → 1~2FPS 키프레임만 레이블링 |
| **HG-DAgger** | 사람이 위험할 때만 개입, 그 순간만 수집 |

교수님 설명: "자율주행 같으면 딱 룰이 있잖아요. 차선의 가운데로 가는 게 가장 좋다라고 하는 룰이 있으면, 내가 롤아웃으로 해서 나온 트래젝토리가 차선의 가장자리로 가고 있어요 라고 하면, 그때의 정답은 차선의 가운데로 가도록 내가 하는 것이 타당하잖아요. 그런 식으로 액션을 레이블링 하는 거예요."

**DAgger vs BC 핵심 차이:**
BC는 전문가가 방문한 스테이트만 학습 → DAgger는 내가 실제로 방문하게 될 스테이트에 대해서도 레이블링을 받음 → training/evaluation 분포 일치 달성.

---

## 7. DART ★★

**교수님의 설명 흐름:**
"다트라고 하는 것도 되게 뭐랄까 조금 참신한 아이디어라고 저는 생각이 듭니다. 어려운 내용은 아니고요." On-policy 방식(DAgger)의 두 가지 한계를 지적하며 등장: (1) 매 rollout마다 피드백 필요, (2) 안전 문제.

**핵심 개념 (강의자료 기준):**
- **DART**: Disturbances for Augmenting Robot Trajectories
- Michael Laskey 외, UC Berkeley, CoRL 2017
- 데이터 **수집 시** 노이즈를 주입해서 분포를 넓힘
- 학습 폴리시가 나중에 방문할 스테이트를 미리 커버하게 함

**DART의 핵심 아이디어 (교수님 설명):**

기존 방식: 전문가가 시연 → 그 트래젝토리대로 이동

DART: 전문가가 "이쪽으로 가라"고 액션을 주면, **실제로는 노이즈를 섞어서** 다른 방향으로 이동 → 전문가가 그 벗어난 상황에서 복구하는 액션도 자동으로 수집됨

> "전문가가 이렇게 가라고 레버를 올렸어요. 그러면 현재 스테이트와 현재 액션을 다 저장합니다. 이 스테이트에서 이 액션이 답이구나. 근데 실제로 이쪽 방향으로 안 가고 거기에 노이즈를 섞어서 랜덤으로 가는 거예요. 전문가가 시연할 때 그걸 벗어난 상황을 일부러 만들어낸 거고요."

> "내가 이렇게 가라고 하면 옆으로 가면 그때 다시 복구하려고 또 액션을 줄 거 아닙니까? 이런 식으로 전문가의 스테이트에 대한 전문가의 액션이 정답으로 기록은 하지만, 실질적으로 스테이트에서 전문가의 액션을 줬을 때 가는 거는 랜덤으로 간다."

**강조 포인트:** ★★ — "참신하다"고 언급. 시간 비중은 DAgger보다 적음. 로봇 오브젝트 피킹 태스크에 적용한 시연 영상을 보여줌.

---

## 8. Behavioral Cloning Summary ★★

**핵심 정리 (강의자료 기준):**

| 구분 | 내용 |
|------|------|
| **장점** | Simple and efficient |
| **단점** | Distribution mismatch (train vs test), No long-term planning |
| **권장 상황** | 1-step deviation이 크게 영향 없을 때, 전문가 트래젝토리가 전체 state space를 커버할 때 |
| **비권장 상황** | 1-step deviation이 catastrophic error로 이어질 때, 장기 목표 최적화가 필요할 때 |

**교수님의 표현:**
> "간단하다 그리고 효과적이다라고 하는 거죠. 단점은 뭐냐, 일단 train-test distribution이 다르게 되면 성능이 확 떨어진다. 그래서 길게 못한다."

> "언제 쓰냐? 그냥 딱 한 스텝 뭔가를 해야 될 때. 어떤 스테이트에 대해서 내가 액션을 한 번 내면 딱 문제가 끝날 때. 한 스텝의 차이가 별로 큰 영향을 미치지 않아요, 그게 누적이 되니까 문제인 거잖아요."

---

## 9. LLM과 Imitation Learning: SFT와 RLHF ★★

**교수님의 설명 흐름:**
"원래 수업 자료는 이 정도였는데 제가 요즘에 조금 많이 나오는 내용에서 추가를 했어요. LLM이라는 게 티가 너무 나서." 직접 추가한 내용임을 명시. IL과 LLM 훈련이 같은 원리임을 보여주는 응용 파트.

**핵심 연결: SFT = Behavioral Cloning on Text**

| 항목 | Robot BC | LLM SFT |
|------|---------|---------|
| State $s$ | camera/sensor frame | prompt + tokens so far |
| Action $a$ | steering/torque | next token |
| Expert $\pi^*$ | human operator | human demonstrator |
| Data $D$ | recorded trajectories | prompt-response pairs |
| Objective | $\min E[L(a^*, \pi_\theta(s))]$ | $\min -\Sigma \log \pi_\theta(a^*\|s)$ (cross-entropy) |

**교수님의 표현:**
> "어떤 LLM이 있는데, SFT는 그 섬 언어를 내가 이렇게 입력을 줬으니까 이렇게 대답을 해야 돼라고 인풋과 출력을 같이 주는 거예요. 비헤이비어 클로닝이랑 같죠. LLM을 폴리시 모델로 본다면 BC와 같다라고 볼 수 있겠죠."

**SFT와 RLHF의 차이 (교수님이 명시적으로 헷갈린다고 주의 환기):**

> "SFT와 RLHF를 헷갈리시는 분들이 많아요. 차이가 뭐냐면, SFT는 내가 이 프롬프트가 들어갔을 때 이 답이 나와야 돼라고 **정해놓은** 겁니다. 근데 RLHF는 답은 모델이 내놓고, 모델이 만약에 3개의 답을 내놨어요. 그러면 내가 1등은 1번, 2등은 3번, 3등은 2번 이렇게 답을 **평가**하는 거예요. 답을 어떤 답을 내야 된다라고까지 강제하지는 않는다는 거예요."

**RLHF 3단계 파이프라인 (강의자료 기준):**

$$\text{1. SFT} \to \text{2. Reward Model (human rankings)} \to \text{3. RL (PPO)}$$

| 단계 | 내용 |
|------|------|
| **①SFT** | 인간이 쓴 답변으로 BC → 기본 폴리시 |
| **②Reward Model** | 인간이 출력 랭킹 매겨서 리워드 함수 학습 |
| **③RL (PPO)** | 리워드 모델 기준으로 폴리시 최적화 |

**RLHF = DAgger 정신 (교수님 연결):**
> "RLHF 같은 경우에는 DAgger와 비슷하다고 할 수 있겠죠. 내가 일단 답을 내놓고 그 답에 대해서 전문가한테 레이블링을 해달라. 레이블링이 정답인지 아닌지 혹은 점수 — 이걸 할 수 있기 때문에 그런 차이가 있겠다."

**RLVR (e.g., GRPO) — 자동화된 리워드:**
> "아예 그것조차 자동화해서 룰을 만들어서, 코딩이나 수학 같은 경우에는 답이 정해져 있죠. 그런 걸 통해서 리워드를 줄 수 있겠죠. PPO나 GRPO 같은 방식으로 모델을 학습한다."

**IL 관점에서 정리한 LLM 훈련의 spectrum (강의자료 기준):**

```
action-level imitation ←————————————→ preference-level alignment

BC/SFT        DAgger        IRL/GAIL        RLHF
"do exactly"  "here you     "why they       "A is better
              should..."    act"            than B"
```

**BC(SFT)의 한계가 LLM에서도 동일하게 나타남:**
> SFT만으로는 학습한 프롬프트 분포 밖에서 distribution shift 발생. RLHF의 RL 단계는 모델 자신의 출력에서 훈련 — DAgger와 같은 정신.

---

## 마지막 점검

**강의자료와 전사본 어긋난 부분:**
- 강의자료 목차에는 "Direct Policy Learning"이 별도 항목으로 있으나, 교수님은 강의 중 "오늘은 BC만 다룰 예정"이라고 명시. DAgger와 DART가 Direct Policy Learning의 구체적 구현에 해당하는 것으로 이해됨 (충돌 없음, 분류 방식의 차이).

**전사가 불명확해 추정한 부분:**
- "배거/대거/데거" → **DAgger** (Dataset Aggregation)로 교정
- "알빈이라고 하는" → **ALVINN**으로 교정
- "코베리에이트 시프트" → **Covariate Shift**로 교정
- "제너레이티브 어디리시리얼 이니케이션 러닝" → **GAIL** (Generative Adversarial Imitation Learning)로 교정
- "RL VR rd" → **RLVR** (RL from Verifiable Rewards)로 추정 [보충: 코딩/수학 등 검증 가능한 태스크에서 자동 리워드 부여하는 방식]
- "뮤비트리" → **Unitree** (유니트리, 중국 로봇 회사)로 추정
- "비에비어 클로닝/베이비오 클로니" → **Behavioral Cloning**으로 교정

**교수님이 다음 강의 예고한 내용:**
- 다음 주: Imitation Learning 2회차 — Inverse RL 등
- 강의 중 보여준 동영상(400MB+)은 교수님이 업로드 불가하다고 언급 — "간직하시기 바랍니다"
- 로봇 관련 여담: 유니트리 로봇 그리퍼 약 5천만원 + 바퀴 달린 몸통 약 8천만원. 미국/중국에서 로봇 데이터 수집은 아직도 사람이 핵심 리소스. 대형 체육관에 수백 대 로봇을 수백 명이 조작하며 1년치 데이터를 쌓는 방식 사용 중.


