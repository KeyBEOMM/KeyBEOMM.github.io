---
title: "Robotics mobile robot"
description: "Robotics"
author: KEYBEOMM
date: 2026-06-16 00:00:00 +0900
categories: [Robotics]
tags: [robot, mobile robot,wheel locomotion]
math: true
---

# [05/26 — Mobile Robot Kinematics] 로봇공학 강의 정리본

> 슬라이드: `260526_로봇공학.pdf` (63장) / 전사본: `05_26.md`
> 교수: 김종우 교수님
> 강의 핵심 한 줄: **"바퀴가 부과하는 제약(wheel constraint)을 모아서 모바일 로봇의 운동학 모델을 만든다."**

---

## 0. 강의 한눈에 보기

### 슬라이드 Outline(목차) 기준 강의 구조 (p2, p8, p19, p39, p44, p54)
강의는 매 섹션 시작마다 똑같은 Outline 슬라이드를 다시 띄우며 진행됐다. 큰 뼈대:

1. **Introduction** — 모바일 로봇 vs 매니퓰레이터, locomotion이란? (p3–7)
2. **Legged locomotion** — 다리 로봇: 왜 쓰나, 자유도, 밸런스 (p9–18)
3. **Wheeled locomotion** — 바퀴 종류, 구성, holonomic/non-holonomic (p20–38)
4. **Representation of a Mobile Robot** — 좌표계·pose·velocity 표기 (p40)
5. **Kinematic Model of the Wheels** — 바퀴 1개의 제약식 유도 (p41–53) ← **유도 핵심**
6. **Mobile Robot Kinematics** — 제약식을 모아 로봇 모델, 차동 로봇 예제 (p55–62) ← **계산 핵심**

### 도입부 프레이밍
교수님은 이 강의를 **"매니퓰레이터(고정 베이스)와 무엇이 다른가"**로 시작한다. 매니퓰레이터는 환경에 고정(fixed base)인데 모바일 로봇은 고정돼 있지 않고 **locomotion(이동 능력)**을 가진다는 게 출발점. 그리고 이동 능력 때문에 생기는 **non-holonomic 제약**이 매니퓰레이터 운동학과 **가장 크게 다른 점**임을 반복 강조한다 (p41).

> **이번 강의는 두 덩어리다.** 앞 절반(legged/wheeled 개념·바퀴 종류)은 개념·암기 위주, 뒤 절반(p40~62 수식 유도)이 **시험에서 손으로 재현해야 하는 진짜 알맹이**다. 교수님도 "각각의 컨스트레인트를 만들 때 무조건 식을 다 유도해 보았습니다"(32:00경)라며 유도 과정을 칠판처럼 한 단계씩 전개했다.

### 다룬 개념 + 중요도 한눈표

| # | 개념 | 슬라이드 | 등급 |
|---|---|---|---|
| 1 | **Conventional wheel 제약식 유도** (rolling+sliding) | p45–51 | ★★★ |
| 2 | **차동(Differential) 로봇 forward/inverse kinematics** | p56–62 | ★★★ |
| 3 | **Static vs Dynamic balance + Support polygon** | p17 | ★★★ |
| 4 | Robot 표현(inertial/robot frame, pose, velocity, 회전행렬) | p40 | ★★ |
| 5 | Forward vs Inverse kinematics 정의 | p42–43 | ★★ |
| 6 | 바퀴 종류 5가지 (fixed/steered/off-centered/Mecanum/spherical) | p21–30 | ★★ |
| 7 | Holonomic vs Non-holonomic | p35–38 | ★★ |
| 8 | Legged locomotion 개요(자유도, kinematic chain, 제어방식) | p9–16 | ★★ |
| 9 | 휠 모델 3가정 (rigid / vertical / 1 contact point) | p45 | ★★ |
| 10 | Off-centered·Swedish 휠 제약식(결과만) | p52–53 | ★ |
| 11 | In-wheel motor, 휠 구성(2/3/4륜), 서스펜션, 큰 바퀴/토크 | p28,31–34 | ★ |

### 지난/다음 강의 연결
- **지난 강의 연결**: 전사 앞부분(문단형)은 지난 시간 legged robot 내용의 복습으로 시작 → 이번에 wheeled 쪽으로 넘어옴. encoder(조인트 각도)·IMU, skew-symmetric 행렬, 회전행렬은 "앞에서 배웠다"고 전제하고 사용 → **선수지식 가정**.
- **다음 강의 예고**: "6월 11일, 6월 16일 두 번 강의 남았다. **torque 관련 분석, manipulator dynamics(다이나믹) 다룬 뒤** 6/18 목요일 기말." (52:48경)
- **시험 예고**(53:14~끝, ⭐중요): 아래 [기말고사 안내](#기말고사-안내-전사-535620으로-전달) 참고.

---

## 용어 대조표

| 영어 (슬라이드) | 한국어/구어 (전사) | 한 줄 뜻 |
|---|---|---|
| Locomotion | 로코모션 | 한 곳에서 다른 곳으로 이동하는 능력 |
| Legged locomotion | 레그드 로코모션 | 다리 기반 이동 (험지 가능, 에너지 비효율) |
| **Wheeled locomotion** | **(전사 자주 "필드 로코모션")** 휠(드) 로코모션 | 바퀴 기반 이동 (평지 에너지 최고효율) |
| Support polygon | 서포트 폴리곤, 지지 다각형 | 접지점들의 convex hull(볼록껍질) |
| Center of Mass/Gravity (CoM/CoG) | 센터 오브 매스/그래비티 | 무게중심 |
| Static / Dynamic balance | 스태틱/다이나믹 밸런스 | 정적/동적 평형 |
| Gait | 게이트 | 보행 패턴(어느 다리가 언제 뜨고 딛나) |
| Whole-body motion | 홀바디 모션 | 모든 limb(팔·다리)을 종합 제어하는 모션 |
| DOF (degrees of freedom) | 자유도 | 독립 운동 변수 개수 |
| Fixed conventional wheel | 픽스드 컨벤셔널 휠 | 축 고정, 축 회전만 ($\dot\varphi$) |
| Steered standard wheel | 스티어드 스탠다드 휠 | 축 회전 + steering 회전($\dot\beta$) |
| Off-centered orientable wheel | 오프센터드 오리엔터블 휠 | 축에 오프셋 $d$, omnidirectional |
| Mecanum / Swedish wheel | 메카넘/수요(Swedish) 휠 | 롤러 45°/90° 부착, omnidirectional |
| Roller (free to spin) | 롤러 (프리투롤/패시브) | 능동 구동 아님, 자유 회전 |
| Spherical wheel / ball caster | 스피리컬 휠 | 구형 바퀴, 3축 회전 |
| Differential drive | 디퍼런셜 (드라이브) | 좌우 바퀴 속도차로 방향 제어 |
| In-wheel motor | 인휠 모터 | 바퀴 안에 모터 내장, 독립 구동 |
| Holonomic / Non-holonomic | 홀로노믹 / 넌(논)홀로노믹 | 제약 적분가능 / 불가능 |
| Ackermann configuration | 에커만 컨피겨레이션 | 자동차식 조향 구조(non-holonomic) |
| Pose | 포즈 | 위치+자세 $(x,y,\theta)$ |
| Inertial / Robot frame | 이너셜(글로벌)/로봇(R) 프레임 | 고정 좌표계 / 로봇에 붙은 좌표계 |
| Twist (velocity) | 트위스트 | 속도 벡터 $\dot\xi=(\dot x,\dot y,\dot\theta)$ |
| Rotation matrix $R$ | 로테이션 매트릭스 | 좌표계 간 변환 행렬 |
| Skew-symmetric matrix $\hat\omega$ | 스큐 시메트릭 (매트릭스) | 각속도의 외적 연산 행렬 |
| Forward / Inverse kinematics | 폴드/포워드 · 인버스 키네메틱스 | 바퀴속도→로봇속도 / 그 역 |
| Pseudo-inverse (Moore-Penrose) | 수도(슈도) 인버스 | 비정방행렬의 유사역행렬 $A^\#$ |
| Rolling / Sliding constraint | 롤링/슬라이딩 컨스트레인트 | 구름(=구동)/미끄러짐(=옆방향 0) 제약 |

**⚠ 전사 핵심 교정**: 이 전사본은 **"필드 로봇/필드 로코모션"을 거의 항상 "wheeled(휠) 로봇/locomotion"의 뜻으로 쓴다.** 문맥이 바퀴·RPM·평지·에너지효율이면 무조건 휠 로봇이다(슬라이드 제목이 "Wheeled Locomotion"). 본 정리본에서는 모두 **휠(wheeled) 로봇**으로 교정해 적었다. 그 외 교정: 폐허→**페어(pair)**, 식단적→**직관적**, 각대→**값(encoder값)**, 고학년대사기→**공학용 계산기**, 트로퍼스/프로버스→**True/False**, "r 곱하기 8"→**$r\dot\varphi$**, "수율"→**휠**.

---

# Ⅰ. 개념 파트 (개념·암기 위주)

## 1. Legged Locomotion (다리 이동) ★★

**교수님의 설명 흐름** (p9–18, 전사 문단형 line1)
도입에서 치타·도마뱀·뱀 등 **자연 모사(bioinspired)**로 시작 → "자연을 완벽히 모사하긴 어렵고(우리는 link·joint·actuator 기반이라 힘줄 기반 생체를 그대로 못 만든다) 큰 특징만 따온다"고 했다. 그리고 **"가장 에너지 효율적인 이동수단은 바퀴"**, "바퀴는 자연에 없지만 평지에서 에너지 손실을 최소로 줄여준다"는 명제를 던지고, 다리 로봇과 대비시킨다.

**핵심 정의 / 비교** (p9, p20)
- **다리(legged) 로봇**: 장애물·계단·비정형 지형(unstructured terrain) 통과 가능. 단, **에너지 비효율 + 제어 복잡(조인트·링크 많음)**. 접지점(contact points) 사이의 지형은 신경 안 써도 됨.
- **바퀴(wheeled) 로봇**: 평지 에너지 효율 최고, 구조·제어 단순(축 따라 회전만), 보통 밸런스 문제 없음. 단, **장애물 있는 지형은 못 감**.
- **wheeled-legged 로봇**(ETH ANYmal, 2024): 다리 끝에 바퀴 → 평지는 바퀴로 효율적, 계단·턱은 다리로. **두 장점 결합**. (★ 시뮬레이션 학습기반 제어)

**자유도(DOF) — 시험 단골** (p11, p14)
- **4족 로봇(StarlETH 등)**: 다리당 **3 자유도** = ① hip abduction(좌우 벌림) ② hip bending ③ knee bending → 4다리면 **12 DOF**.
- **휴머노이드**: 다리당 보통 **6 자유도** = hip 3 + knee 1 + ankle 2 → 두 다리 **12 DOF**. 여기에 팔(6 또는 redundancy용 7 DOF)×2 + 목·손가락까지 더하면 **26~50+ DOF**로 쉽게 커진다.
  - 교수님 계산 그대로: "팔 7자유도×2=14, +다리 12 = 26자유도, 목·손가락까지 가면 50 몇 자유도."

**핵심: 다리 = serial kinematic chain** (p15)
- 다리 하나를 **base→joint→발목→발끝**으로 이어진 **revolute joint 직렬 체인**으로 보면, 그대로 **매니퓰레이터의 forward/inverse kinematics**로 해석 가능. → "레그도 따져보면 키네마틱 체인(매니퓰레이터)이다."

**제어 방식 2가지** (p16)
1. **Independent(독립)**: 각 다리 따로 제어. 시스템 **redundancy(여유자유도)를 못 살림**. 대신 **제어 쉬움**.
2. **Synchronized(동기화)**: 모든 limb(팔·다리, "림브")를 함께. redundancy 활용, **whole-body motion(홀바디 모션)** 가능. 단 **"floating base" 모델 필요**: $q=(q_{act},\,x_{base})$.
   → 실무에선 동기화로 whole-body motion 제어를 많이 쓴다.

**그림으로 보는 직관** (p10, p18)
- p10: "less stable(멈추면 넘어짐)" ↔ "more stable(멈춰도 안 넘어짐)"을 그림으로 대비. 다리 적을수록 빠른 제어 필요, 다리 많을수록 안정·조정 필요.
- 4족 보행 직관(전사): **대각선 다리끼리 페어(pair)**를 이뤄 움직인다. 1번 페어가 땅 딛는 동안 2번 페어가 공중 이동 → 교대. 6족도 엇갈려 페어. **엇갈린 다리가 페어를 이루면 support polygon이 넓고 좌우 균형이 맞아 더 안정적**.

**⭐ 시험 대비**
- **재현 목표**: 다리 1개를 매니퓰레이터로 보고 DOF 세기(4족=12, 휴머노이드 다리=12). independent vs synchronized 제어 차이 서술.
- **예상 유형**: 개념비교형 / 그림설명형 (True/False 가능).
- **자주 틀리는 지점**: hip abduction을 빠뜨려 다리당 2 DOF로 세는 것. independent 제어가 "redundancy를 못 쓴다"는 점.
- **핵심 구분 암기**: legged(험지O·효율X·복잡) ↔ wheeled(평지효율O·험지X·단순) / independent ↔ synchronized.

---

## 2. Static vs Dynamic Balance & Support Polygon ★★★

**교수님의 설명 흐름** (p17, 전사에서 **두 번 반복** 설명)
"**필연적으로 넘어지지 말아야 한다 — 넘어지지 않는 게 최우선 과제**"라고 강조하며 들어간다. 4족 로봇을 예로 그림을 그려가며 정적/동적 밸런스를 두 번 설명했다(반복=강조 신호).

**핵심 정의** (p17)
- **Support polygon(지지 다각형)**: *접지점들의 convex hull(볼록껍질)*. 슬라이드 정의 그대로: "convex hull of the points in contact with the ground."
- **Static balance(정적 밸런스)**:
  - 밸런스가 **항상** 유지됨.
  - 조건: **무게중심(CoM/CoG)을 수직 투영(projection)했을 때 support polygon 내부**에 있어야 함.
  - **느린 모션(한 번에 다리 1개씩 듦, one leg per time)**.
  - 접지 많아 **마찰↑ → 에너지 비효율**.
- **Dynamic balance(동적 밸런스)**:
  - 시스템이 **계속 움직이는 동안만** 밸런스 유지(**급정지(sudden stop) 시 넘어질 수 있음**).
  - 무게중심이 **support polygon 밖으로 나가도 됨**.
  - **빠른 모션(동시에 여러 다리 공중)**.
  - 마찰↓ → **에너지 효율적**, 단 **제어 복잡**.

**그림으로 보는 직관** (p17 우측 두 그림)
- 위 그림: 정적 — CoG에서 내린 수직선이 빨간 삼각형(support polygon) **안**에 떨어짐 → 안정. (전사: "그래비티 방향으로 중력이 작동하는데 polygon이 받쳐주는 방향이 없으면 그쪽으로 토크가 생겨 넘어진다.")
- 아래 그림: 동적 — CoG가 polygon 가장자리/밖이고 빨간 회전 화살표(넘어지려는 모멘트)가 있지만, 빠르게 움직여 균형을 유지.

**⭐ 시험 대비**
- **재현 목표**: 4족 로봇 그림에 접지점→support polygon 그리고, CoM 투영이 내부/외부인지로 정적 안정 판정. 정적·동적 밸런스 4요소(밸런스유지조건/모션속도/polygon관계/에너지) 비교.
- **예상 유형**: 그림설명형 / 개념비교형 (True/False 단골).
- **자주 틀리는 지점**: support polygon이 "접지점들의 convex hull"이라는 정의. 정적은 CoM이 **반드시 내부**, 동적은 **밖 허용**.
- **핵심 구분 암기**: Static(느림·내부·비효율·항상안정) ↔ Dynamic(빠름·밖허용·효율·움직일때만).

---

## 3. 바퀴 종류 (Types of Wheels) ★★

**교수님의 설명 흐름** (p21–30)
"바퀴 타입도 다양하다"며 가장 단순한 것부터 복잡한 것 순으로 5종을 소개. 각 바퀴가 **어떤 회전 자유도를 갖느냐**가 핵심.

| 바퀴 | 자유도 | 특징 | 슬라이드 |
|---|---|---|---|
| **① Fixed conventional** | 축 회전 $\dot\varphi$만 | 축이 바디에 **고정**. 가장 단순 | p21 |
| **② Steered standard** | $\dot\varphi$ + steering $\dot\beta$ | 축 방향도 회전 가능(조향). | p21 |
| **③ Off-centered orientable** | $\dot\varphi$ + $\dot\beta$, 회전축과 바퀴중심에 **오프셋 $d$** | omnidirectional, 어느 방향이든 이동 | p22 |
| **④ Mecanum / Swedish** | 바퀴축 $\dot\varphi$(능동) + 롤러(수동) | 롤러 45°(Mecanum)/90°(Swedish), omnidirectional | p23–27 |
| **⑤ Spherical (ball caster)** | 3축 회전 | 구형, 능동/수동 가능 (BB-8) | p29–30 |

### Mecanum wheel 깊이 보기 ★★ (p23–27) — 기하 직관이 시험감
**원리**(p24, 영상 인용 포함): 메카넘 휠은 림 둘레에 **롤러를 45°로 비스듬히** 단 바퀴(tireless wheel). 롤러는 **free-to-spin(패시브)**. 일반 바퀴는 굴릴 때 지면 접점에서 **tangent(앞뒤) 방향 힘**만 주지만, 45° 롤러 때문에 **롤러 축 방향(수직 성분)으로 힘이 전달** → 바퀴 1개의 구동력이 **45° 방향**으로 나온다.

**4바퀴 조합으로 omnidirectional**(p25, 전사 영상):
- 대각선으로 마주보는 바퀴를 **하나의 페어**로, 한 페어는 롤러 45°, 다른 페어는 **반대방향 45°**로 배열.
- **전진(Y)**: 네 바퀴 같은 방향 → Y성분 더해지고 X성분 상쇄.
- **횡이동(X, lateral)**: 앞왼/뒤오른은 앞으로, 앞오른/뒤왼은 뒤로 → X성분 더해지고 Y상쇄 → **옆으로 쭉**.
- **대각선**: 한 페어만 구동.
- **제자리 회전**: 적절한 조합.
→ "기존 휠 로봇이 trajectory에 의존(평행주차 시 앞뺐다 꺾었다)하는 한계를 극복." 좁은 공간(물류창고·공장)에서 강력.

**단점**(p26, 한글 슬라이드): 구조 복잡(제작·유지비↑), 접점이 롤러 1~2개뿐이라 **마찰↓·미끄러지기 쉬움**(고속·고하중 불리), 하중 지지력 약함. → 롤러를 원기둥 대신 **가운데가 볼록**하게 만들어 접촉면↑·진동 완화.

**활용**(p27): 공장 AGV/AMR, **Airbus A380 제작현장(KUKA OmniMove)** — 거대 동체를 좁은 조립장에서 운반. in-wheel servo motor와 조합이 효과적.

### In-wheel motor (p28) ★
일반차: 엔진→축→두 바퀴(같이 회전). **인휠 모터**: 바퀴 안에 모터 → **좌우 독립 RPM 제어** 가능 → 제어 자유도↑. 장점: transmission 손실↓(효율↑), 독립 제어, 회생제동. 단점: 모터 4개라 **비용↑**, 노출되어 안전 이슈, 서스펜션 구성·고속 기어링 어려움.

### 휠 구성(configurations)과 특성 (p31–34) ★
- **2륜**: bicycle(앞바퀴만 조향, 뒤는 축 고정), differential(Segway식, 좌우 속도차), Cye.
- **3륜**: 패시브 휠 조합, 또는 **원에 120° 배열**해 각 바퀴 X·Y 성분 조합 → omnidirectional.
- **4륜**: 두 바퀴를 한 모터에 물릴지 독립 구동할지 / 휠 타입(Mecanum, 오프셋) 선택. (CMU Uranus, Nomad XR4000)
- **특성**(p34): 3바퀴면 정적 밸런스 보장(삼각형 polygon 안에 CoM), 2바퀴는 CoM이 축 아래면 가능, 4륜↑은 안정↑지만 **서스펜션 필요**. **큰 바퀴**는 장애물 극복 가능하나 **더 큰 토크·기어박스 필요**(토크↑↔속도↓ trade-off).

**⭐ 시험 대비**
- **재현 목표**: 5종 바퀴를 자유도($\dot\varphi$, $\dot\beta$, 오프셋, 롤러)로 구분 / Mecanum 45° 롤러가 왜 omnidirectional인지 X·Y 성분 분해로 설명·도시.
- **예상 유형**: 개념비교형 / 그림설명형.
- **자주 틀리는 지점**: fixed($\dot\varphi$만) vs steered($\dot\varphi+\dot\beta$) vs off-centered(+오프셋$d$) 구분. Mecanum 롤러는 **수동(free-to-spin)**, 바퀴 축만 능동.
- **핵심 구분 암기**: Mecanum(45°, 일반형) ↔ Swedish/omni(90°). 둘 다 omnidirectional이나 롤러 각도가 다름.

---

## 4. Holonomic vs Non-holonomic ★★

**교수님의 설명 흐름** (p35–38)
바퀴 구성 대부분이 **non-holonomic**임을 강조하며 평행주차 예시로 직관을 잡아줌.

**정의** (p35–37)
- **Holonomic system**: 제약을 **적분 가능(integrable)**.
- **Holonomic robot**: 속도 $v(t)$가 적분 가능 — **위치 $(x,y)$와 자세 $\theta$에만 의존하는 trajectory $s(t)$가 존재**. 적분가능 조건(슬라이드): $s=s(x,y,\theta)$이고 2계 편미분 교환($\partial^2 s/\partial x\partial y=\partial^2 s/\partial y\partial x$ 등).
  - 특징: **pose가 velocity에 제약받지 않음**, **어느 방향으로든 즉시 이동** → **omnidirectional 로봇**(Mecanum, 120°배열 등)이 여기 해당. *특수한 경우.*
- **Non-holonomic system**: 제약을 **적분 불가능**.
- **Non-holonomic robot**: $v(t)$가 적분 불가, 위치·자세에만 의존하는 $s(t)$가 없음. 도달 가능한 **속도(방향)에 제약**. 예: **Ackermann(자동차)**. *일반적인 휠 로봇.*

**그림으로 보는 직관** (p38) — 시험 단골
- 두 경로 모두 **좌우 바퀴 이동거리(travelled distance)가 동일**: $s_{1R}=s_{2R},\ s_{1L}=s_{2L}$.
- 그런데 **최종 위치는 다름**: $(x_1,y_1)\ne(x_2,y_2)$.
- **결론(슬라이드)**: "**각 바퀴가 이동한 거리만으로는 final pose를 계산할 수 없다**. 운동의 **시간적 전개(temporal evolution)**까지 알아야 한다." → 그래서 **위치가 아니라 속도(velocity)를 다루는 differential kinematics**가 필요(p41).

**⭐ 시험 대비**
- **재현 목표**: holonomic↔non-holonomic 정의(적분가능성), 특징 3개씩 서술. p38 두-경로 그림으로 "거리만으론 pose 결정 불가" 설명.
- **예상 유형**: 개념비교형 (True/False), 그림설명형.
- **자주 틀리는 지점**: omnidirectional=holonomic은 **특수**, 일반 휠 로봇(자동차)은 **non-holonomic**. "제약이 적분 가능한가"가 판별 기준.
- **핵심 구분 암기**: Holonomic(omni·즉시 전방향·pose가 v에 무관) ↔ Non-holonomic(자동차·평행주차·방향 제약).

---

# Ⅱ. 유도 파트 (시험 핵심 — 손으로 재현)

## 5. 모바일 로봇의 표현 (Representation) ★★

**교수님의 설명 흐름** (p40, 전사 09:14~10:57)
유도에 들어가기 전 **두 좌표계**를 세운다. 매니퓰레이터와 달리 모바일 로봇은 움직이므로 **글로벌 프레임 + 로봇에 붙은 프레임**이 둘 다 필요.

**핵심 정의 / 표기** (p40, 슬라이드 그대로)
- **Inertial(글로벌·절대·고정) reference frame $\{I\}$**: 외부에 정지. 축 $X_I, Y_I$.
- **Robot's reference frame $\{R\}$**: 로봇 중심에 붙어 **로봇과 함께 이동·회전**. 진행(바라보는) 방향이 $X_R$, 좌측이 $Y_R$. (전사: 빨간색이 R, 파란색이 I 프레임.)
- **Robot position**: $\,^I\mathbf{r}_{OR}=\begin{bmatrix}x\\y\\0\end{bmatrix}$, **orientation** $\theta$.
- **Pose** (2D는 한 벡터로): $\,^I\xi=\begin{bmatrix}x\\y\\\theta\end{bmatrix}$
- **Velocity(twist)**: $\,^I\dot\xi=\begin{bmatrix}\dot x\\\dot y\\\dot\theta\end{bmatrix}$, 그리고 프레임 변환: $\boxed{\,^R\dot\xi = \,^RR_I\,^I\dot\xi}$
  - $^RR_I$: 글로벌(I)에서 본 속도를 로봇(R) 프레임에서 본 속도로 바꾸는 **회전행렬**.
- **로봇 프레임에서 자기 위치**: 항상 $(0,0,0)$ (전사 10:28: "로봇과 같이 움직이니 R프레임에서 로봇 위치는 0,0,0").

**⭐ 시험 대비**
- **재현 목표**: pose·twist 벡터 표기, $^R\dot\xi=\,^RR_I\,^I\dot\xi$ 변환 쓰기. 2D 회전행렬 $^RR_I=\begin{bmatrix}\cos\theta&\sin\theta&0\\-\sin\theta&\cos\theta&0\\0&0&1\end{bmatrix}$ [보충: 2D 표준형] 직접 쓰기.
- **자주 틀리는 지점**: 변환 방향($^RR_I$ vs $^IR_R$)과 전치 혼동. 위·아래 첨자(어느 프레임→어느 프레임) 정확히.

---

## 6. Differential Kinematics 정의 ★★ (p41–43)

**왜 differential인가** (p41)
- 운동학은 **모터(바퀴)와 Cartesian 공간(위치·자세)을 연결**.
- **non-holonomic 제약 때문에** 모바일 로봇에선 **differential kinematics(속도 관계)**를 쓴다 — 위치가 아니라 **속도 변환**을 다룬다.
- 일반적으로 **위치 관계는 못 얻는다(속도 적분 불가)**. → **매니퓰레이터 운동학과의 핵심 차이.**

**Forward / Inverse** (p42)
- **Forward kinematics**: 바퀴속도 → 로봇속도. $\,^I\dot\xi=f(\dot\varphi_1,\cdots,\dot\varphi_n,\ \beta_1,\cdots,\beta_m,\ \dot\beta_1,\cdots,\dot\beta_m)$
- **Inverse kinematics**: 로봇속도 → 바퀴속도. $[\dot\varphi_i,\cdots,\dot\beta_i]^T=f(\dot x,\dot y,\dot\theta)$
- 기호: $\dot\varphi_i$ = 바퀴 축 회전속도, $\dot\beta_i$ = 각 바퀴의 **steering 속도**. $n$ 바퀴, $m$ orientable 바퀴.

**목표/방법** (p43)
- 목표: 설계·제어를 위한 로봇 거동 기술.
- 방법: **바퀴 $M$개 각각이 운동에 제약(0/1/2개)을 부과** → 이 제약들이 로봇 운동학을 정의. → **"바퀴가 부과하는 제약을 분석해야 운동을 이해한다."**

---

## 7. 【핵심 유도】 Conventional Wheel의 제약식 ★★★

> **이 절이 시험 손유도의 토대.** 교수님: "각각의 컨스트레인트를 만들 때 **무조건 식을 다 유도**해 보았습니다"(32:00). 단계별로 그대로 보존한다.

### (0) 휠 모델 3가정 (p45)
**Ideal wheel 가정**:
1. **The structure is rigid** (바퀴 강체, 변형 없음).
2. **The plane of the wheel is vertical** (바퀴면 수직, 기울지 않음).
3. **There is only 1 contact point** (지면과 접점 1개).

이 가정에서 **두 제약(constraints)**:
1. **바퀴 평면 위: pure roll(순수 구름), no sliding** (앞뒤로 구름만).
2. **바퀴에 수직인 평면: there is no motion** ($v=0$, 옆방향 미끄러짐 없음).

(전사 보충: 드리프트처럼 미끄러지는 건 일단 배제. 평지·수직·단일접점의 단순 모델.)

### (1) 휠 프레임에서의 제약 (p46)
바퀴 프레임 $\{W\}$: **$X_W$ = 바퀴 축 방향(옆방향)**, **$Y_W$ = 구르는 방향(앞방향)**. 그러면:
- $v_x=0$ (옆방향 속도 0, 슬라이딩 없음)
- $v_y=r\dot\varphi$ (구름 속도 = 반지름 × 바퀴 각속도)

$$\boxed{\,^W\dot{\mathbf r}=\begin{bmatrix}0\\ r\dot\varphi\\ 0\end{bmatrix}}\qquad(\text{wheel constraint})$$

### (2) 바퀴 위치의 분해 (p47)
바퀴 중심 $W$까지의 위치를 **로봇 중심 경유**로 분해(삼각형 벡터합):
$$\,^I\mathbf{r}_{OW}=\,^I\mathbf{r}_{OR}+\,^I\mathbf{r}_{RW}=\,^I\mathbf{r}_{OR}+\,^IR_R\,^R\mathbf{r}_{RW}$$
여기서(슬라이드 우측 그림, $R$에서 $W$까지 거리 $l$, 각 $\alpha$):
$$\,^I\mathbf{r}_{OR}=\begin{bmatrix}x\\y\\0\end{bmatrix},\qquad \,^R\mathbf{r}_{RW}=\begin{bmatrix}l\cos\alpha\\ l\sin\alpha\\ 0\end{bmatrix}$$
- **직관**(전사 19:48~): 바퀴 위치를 글로벌에서 바로 다루기보다, **로봇 중심에서 바퀴까지(R프레임)는 항상 일정**하므로 "로봇중심 위치 + (R프레임에서 본 바퀴 상대위치)"로 쪼개면 훨씬 편하다.

### (3) 시간 미분 → 속도 (p48–49) — **한 항이 사라지는 이유가 포인트**
양변 시간 미분:
$$\,^I\dot{\mathbf r}_{OW}=\,^I\dot{\mathbf r}_{OR}+\,^I\dot R_R\,^R\mathbf{r}_{RW}+\underbrace{\,^IR_R\,^R\dot{\mathbf r}_{RW}}_{=0}$$
- **마지막 항 소거**(p48 동그라미): $^R\mathbf{r}_{RW}$(로봇중심→바퀴, R프레임 기준)는 **시간에 따라 안 변함(constant)** → 미분 0. (전사 24:41: "자동차 중심에서 바퀴 중심까지는 늘 일정하니 이 항은 날아간다.")

회전행렬 미분 공식 $\,^I\dot R_R=\,^I\hat\omega_R\,^IR_R$ 대입:
$$\,^I\dot{\mathbf r}_{OW}=\,^I\dot{\mathbf r}_{OR}+\,^I\hat\omega_R\,^IR_R\,^R\mathbf{r}_{RW}=\,^I\dot{\mathbf r}_{OR}+\,^I\hat\omega_R\,^I\mathbf{r}_{RW}$$
여기서 각속도와 **skew-symmetric 행렬**(p49):
$$\,^I\omega_R=\begin{bmatrix}0\\0\\\dot\theta\end{bmatrix},\qquad \hat\omega=\begin{bmatrix}0&-\omega_z&\omega_y\\ \omega_z&0&-\omega_x\\ -\omega_y&\omega_x&0\end{bmatrix}$$
- **직관**: 로봇이 $\dot\theta$로 회전 → R프레임 자체가 I에 대해 $\dot\theta$로 회전. 그래서 바퀴 속도에 회전 기여항 $\hat\omega\times r$이 붙는다. (skew-symmetric은 "앞에서 배웠다"고 전제 — 외적을 행렬곱으로.)

### (4) 휠 프레임으로 회전 + 제약 적용 (p50–51) — **결론식**
양변에 $^WR_I$ 곱해 **휠 프레임**에서 보면, 좌변은 wheel constraint $[0,\,r\dot\varphi,\,0]^T$. $\alpha,\beta,l$로 전개하면:

$$\boxed{\begin{bmatrix}0\\ r\dot\varphi\\ 0\end{bmatrix}=\begin{bmatrix}\cos(\alpha+\beta)&\sin(\alpha+\beta)&l\sin\beta\\ -\sin(\alpha+\beta)&\cos(\alpha+\beta)&l\cos\beta\\ 0&0&1\end{bmatrix}\,^RR_I\,\underbrace{\,^I\dot\xi}_{[\dot x,\dot y,\dot\theta]^T}}$$

여기서 위 두 행을 풀면 **두 제약식**:

**① Rolling constraint (운동에 평행 = 구름 = 구동)** ← 슬라이드 "parallel to the motion"
$$\boxed{[\,-\sin(\alpha+\beta)\quad \cos(\alpha+\beta)\quad l\cos\beta\,]\,^RR_I\,^I\dot\xi\;-\;r\dot\varphi=0}$$

**② Sliding constraint (운동에 수직 = 옆방향 속도 0)** ← 슬라이드 "perpendicular to the motion"
$$\boxed{[\,\cos(\alpha+\beta)\quad \sin(\alpha+\beta)\quad l\sin\beta\,]\,^RR_I\,^I\dot\xi=0}$$

- 세 번째 행(0=0)은 **자동 만족**(전사 29:46).
- **의미**(전사 31:08): ①은 "바퀴가 구를 때 앞방향 속도 = $r\dot\varphi$", ②는 "바퀴면에 수직(옆) 방향 속도 = 0".

**⭐ 시험 대비 — 이 절이 최우선**
- **재현 목표**: 위치 분해(p47)→시간 미분에서 **상수항 소거**(p48)→skew-symmetric로 회전항(p49)→휠 프레임 변환→**rolling/sliding 두 제약식**(p51)까지 **백지에서 전개**. 교수님이 "컨벤셔널 식만 잘 유도하면 응용문제 다 푼다"고 명시.
- **예상 유형**: **유도형**(과정 전개) + 계산형(특정 $\alpha,\beta,l$ 대입).
- **자주 틀리는 지점**: ① 미분에서 $^R\mathbf{r}_{RW}$가 상수라 한 항 사라지는 근거 누락. ② rolling/sliding 두 식의 첫 두 성분 부호($-\sin$ vs $\cos$) 뒤바꿈. ③ $l\sin\beta$/$l\cos\beta$ 항 위치. ④ 위/아래 첨자($^WR_I=\,^WR_R\,^RR_I$) 표기.
- **핵심 구분 암기**: rolling(평행, $=r\dot\varphi$) ↔ sliding(수직, $=0$).

### (5) 다른 바퀴의 제약식 — 결과만 (p52–53) ★
교수님: "컨벤셔널만 잘 유도하면 나머지는 결과만 알면 된다. 여기 d나 $\gamma$ 항만 추가됨을 알아두라."

**Off-centered orientable wheel**(오프셋 $d$, p52):
- 평행: $[-\sin(\alpha+\beta)\ \cos(\alpha+\beta)\ l\cos\beta]\,^RR_I\,^I\dot\xi-r\dot\varphi=0$ (컨벤셔널과 동일)
- 수직: $[\cos(\alpha+\beta)\ \sin(\alpha+\beta)\ d+l\sin\beta]\,^RR_I\,^I\dot\xi+d\dot\beta=0$ (**$d$, $d\dot\beta$ 추가**)

**Omnidirectional / Swedish wheel**(롤러 각 $\gamma$, p53):
- 평행: $[\sin(\alpha+\beta+\gamma)\ -\cos(\alpha+\beta+\gamma)\ -l\cos(\beta+\gamma)]\,^RR_I\,^I\dot\xi-r\dot\varphi\cos\gamma=0$
- 수직: $[\cos(\alpha+\beta+\gamma)\ \sin(\alpha+\beta+\gamma)\ l\sin(\beta+\gamma)]\,^RR_I\,^I\dot\xi-r\dot\varphi\sin\gamma-r_{sw}\dot\varphi_{sw}=0$ (**롤러 항 $r_{sw}\dot\varphi_{sw}$ 추가**)

---

## 8. 【핵심 계산】 Differential Robot 예제 ★★★

> **시험에 변형되어 나온다고 명시**된 worked example (p56–62). 교수님: LMS(주차학습)의 forward/inverse 풀이에서 "한 문제 정도 변형되어 나올 수 있으니 꼭 복습".

### (0) 시스템 설정 (p56–57)
- **2개의 fixed conventional wheel**, 로봇 프레임 $\{R\}$은 **두 바퀴 중점**에 위치($X_R$ 전진, $Y_R$ 좌측).
- 앞쪽 동그란 바퀴는 **passive(무게중심 안정용)** — 모터 없음. 구동은 **좌·우 두 바퀴의 RPM 차이**(differential).
  - (전사: 두 바퀴를 같은 속도로만 돌리면 전진/후진만 가능, **방향 전환하려면 좌우 속도차 필요**.)
- 파라미터: 바퀴 반지름 $r$, 중심에서 각 바퀴까지 거리 $b$ (즉 **트랙폭 = $2b$**).
- 바퀴별 $\alpha,\beta,l$ (p57, fixed라 $\beta=0$):
  - **Right wheel**: $\alpha=-90^\circ,\ \beta=0,\ l=b$
  - **Left wheel**: $\alpha=-90^\circ,\ \beta=0,\ l=-b$
  - (전사 38:29: 왼쪽 휠은 $\alpha=-90^\circ$ 두고 $l=-b$로 잡거나, $\alpha=+90^\circ$에 $l=b$로 잡거나 **결과는 동일**.)

### (1) 각 바퀴 제약식 대입 (p58)
$\alpha+\beta=-90^\circ$이므로 $\sin(-90^\circ)=-1,\ \cos(-90^\circ)=0,\ \cos\beta=1,\ \sin\beta=0$. 7절의 두 제약에 대입(로봇 프레임에서 $^RR_I\,^I\dot\xi=\,^R\dot\xi$로 둠):
- 일반형: rolling $\to[1,\ 0,\ l]\,^R\dot\xi=r\dot\varphi$, sliding $\to[0,\ -1,\ 0]\,^R\dot\xi=0$

- **Right** ($l=b$): $[1,0,b]\,^R\dot\xi=r\dot\varphi_r$, $\quad[0,-1,0]\,^R\dot\xi=0$
- **Left** ($l=-b$): $[1,0,-b]\,^R\dot\xi=r\dot\varphi_l$, $\quad[0,-1,0]\,^R\dot\xi=0$

### (2) 행렬로 묶기 (p59)
4개 식을 모으면 $A\,^R\dot\xi=B\dot\varphi$:
$$\underbrace{\begin{bmatrix}1&0&b\\ 1&0&-b\\ 0&-1&0\\ 0&-1&0\end{bmatrix}}_{A\,(4\times3)}\,^R\dot\xi=\underbrace{\begin{bmatrix}r&0\\ 0&r\\ 0&0\\ 0&0\end{bmatrix}}_{B\,(4\times2)}\begin{bmatrix}\dot\varphi_r\\\dot\varphi_l\end{bmatrix}$$

### (3) Forward kinematics (바퀴속도 → 로봇속도) (p60)
$A$가 정방행렬이 아니므로 **Moore-Penrose pseudo-inverse** $A^\#=(A^TA)^{-1}A^T$ 사용:
$$\,^R\dot\xi=A^\#B\dot\varphi=\begin{bmatrix}\tfrac{r}{2}&\tfrac{r}{2}\\ 0&0\\ \tfrac{r}{2b}&-\tfrac{r}{2b}\end{bmatrix}\begin{bmatrix}\dot\varphi_r\\\dot\varphi_l\end{bmatrix}$$
$$\boxed{\,^R\dot x=\frac{r}{2}(\dot\varphi_r+\dot\varphi_l),\qquad \dot y=0,\qquad \dot\theta=\frac{r}{2b}(\dot\varphi_r-\dot\varphi_l)}$$
- **의미**: 전진속도 = 두 바퀴속도 평균×$r$. **$\dot y=0$**은 옆방향 속도 0 — non-holonomic 제약을 그대로 확인(전사 48:11: "제대로 풀면 $\dot y$가 0이 나온다"). 회전속도 = 속도차 ÷ $2b$ × $r$. ($\dot\varphi_r>\dot\varphi_l$이면 $\dot\theta>0$ → 좌회전, 전사의 "오른쪽이 빠르면 좌회전"과 일치.)

### (4) Inverse kinematics (로봇속도 → 바퀴속도) (p61)
$B$도 비정방 → $B^\#=(B^TB)^{-1}B^T$:
$$\dot\varphi=B^\#A\,^R\dot\xi=\begin{bmatrix}\tfrac1r&0&\tfrac br\\ \tfrac1r&0&-\tfrac br\end{bmatrix}\begin{bmatrix}\,^R\dot x\\ \dot y\\ \dot\theta\end{bmatrix}$$
$$\boxed{\dot\varphi_r=\frac1r(\,^R\dot x+b\dot\theta),\qquad \dot\varphi_l=\frac1r(\,^R\dot x-b\dot\theta)}$$

### (5) 결과 요약 (p62)
| | 식 |
|---|---|
| **Forward** | $^R\dot x=\frac r2(\dot\varphi_r+\dot\varphi_l)$, $\ \dot\theta=\frac{r}{2b}(\dot\varphi_r-\dot\varphi_l)$ |
| **Inverse** | $\dot\varphi_r=\frac1r(^R\dot x+b\dot\theta)$, $\ \dot\varphi_l=\frac1r(^R\dot x-b\dot\theta)$ |

기호: $\dot\varphi_r,\dot\varphi_l$ 우/좌 바퀴 각속도, $r$ 바퀴 반지름, $^R\dot x$ 로봇 x선속도, $\dot\theta$ 로봇 각속도, $b$ 중심~바퀴 거리.

**⭐ 시험 대비 — 계산 핵심**
- **재현 목표**: $\alpha=-90^\circ,\beta=0,l=\pm b$ 대입 → $A,B$ 구성 → **pseudo-inverse로 forward/inverse 둘 다 유도** → 최종 4식. 숫자 주면(예: 바퀴 20 RPM, 또는 "x로 2 m/s 전진 + 초당 2° 회전") **수치 계산**.
- **예상 유형**: 유도형 + 계산형. (LMS 풀이 변형 1문제 예고.)
- **자주 틀리는 지점**: ① $A,B$ 행/열 배치 실수. ② pseudo-inverse는 비정방이라 필요 — 일반 역행렬과 혼동. ③ $\dot y=0$ 누락. ④ $2b$ vs $b$ (트랙폭 vs 반폭). ⑤ 좌/우 부호($+b\dot\theta$ vs $-b\dot\theta$).
- **핵심 구분 암기**: Forward($A^\#B$, 바퀴→로봇) ↔ Inverse($B^\#A$, 로봇→바퀴). $A^\#=(A^TA)^{-1}A^T$, $B^\#=(B^TB)^{-1}B^T$.

---

## 결론 (p63)
- 바퀴는 운동에 **제약(restriction)**을 부과한다.
- 로봇의 운동학 모델은 **바퀴 제약들로부터** 얻는다.
- **non-holonomic 제약 때문에 운동학은 속도(velocity)를 다룬다.**
- 운동학 모델은 **분석·운동제어(motion control)**에 쓰인다. (전사 54:24: 3차 가속도까지 갈 필요 없이 모터와 연결된 바퀴 속도로 전체 모션을 분석·제어.)

---

## 기말고사 안내 (전사 53:56~56:20으로 전달) ⭐
- **일시·장소**: **6월 18일(목) 수업 시간, 이 강의실**에서 시행.
- **남은 강의**: 6월 11일·6월 16일 두 번 → **torque 분석, manipulator dynamics(다이나믹)** 다룬 뒤 시험 범위 포함.
- **유형**: **중간고사와 비슷**. **공학용 계산기 사용 가능**. 개념은 **True/False 문제**(전사 "트로퍼스/프로버스" → T/F로 교정).
- **언어**: **문제 전부 영어 출제** → 영어 용어·문제 이해 숙달 필수(용어 대조표 활용).
- **꼭 복습**: **LMS(주차학습)에 올라간 강의의 forward/inverse kinematics 풀이** — 거기서 **변형 문제 1개 정도** 출제 가능.

---

## 곁가지·여담 모음 (★ — 맥락·재미용, 시험 비핵심)

- **로봇 학회/전시 참관기**(전사 00:11~07:00): 오스트리아 **빈(Vienna)**에서 열린 로봇 학회/전시 참관 — 최신·미출시 로봇 다수. dexterous(덱스터러스) 핸드 연구 활발: **Sharpa(중국)** 로봇 딜러(카드 나눠주기), **Tesollo(한국)** 핸드, 손끝 force 센서로 사과 깎기(in-hand manipulation), 축구 슈팅 데모. **내년엔 한국 COEX에서 처음 개최**(5월말~6월초, 약 1만 명 규모). [전사 추정: 학회명 "아이크라/아이카" → **ICRA** 추정이나 전사 불명확].
- **ETH 레그드 로봇**: StarlETH(2013, ICRA 데모) — 다리당 3 DOF, **Maxon(스위스) 모터**, 발끝 high-damping·스프링으로 충격 흡수, **최적제어 기반**(상황별 모델). 2024년 **wheeled-legged(ANYmal류)**는 **시뮬레이션 학습 기반** 제어 + LiDAR로 장애물 회피·계단. **Marco Hutter 교수** 연구실(동굴 탐사 등). IMU + joint encoder를 입력으로 수천~수만 번 학습.
- **Festo**(네덜란드/독일 공압 회사): bio-inspired 로봇(날아다니는 것 포함) 유튜브 다수.
- **Sony Aibo / Asimo**: 과거 일본이 휴머노이드·애완견 로봇 선도 → 투자 부족으로 지금은 미·중이 압도적 선도.
- **Segway 비화**: 미국서 큰 인기·소송 → 중국 회사가 **적대적 M&A로 Segway를 인수**해 해결. 하드웨어 스타트업의 복제·저가 도전 + 지재권 교훈.
- **한국타이어 광고 + 이순걸 교수 연구**: 구형(spherical) 휠 로봇으로 사람 태우고 omnidirectional 고속 — TV 광고화. 단 **빗물 위 주행 등은 실제 불가, CG**. 사람 태운 고속 omni는 지금도 안정성 난제.
- **Airbus A380 + KUKA OmniMove**(p27): Mecanum 18개(한쪽 9개) 배치, 거대 동체를 좁은 조립장에서 운반.
- **서스펜션 여담**(p34): 렉서스=부드러움, BMW=노면감(딱딱)으로 운전 재미 — "무조건 부드럽다고 좋은 건 아니다." 오프로드 지프=큰 바퀴(토크↑·기어비↑).

---

## 마지막 점검

**ASR로 추정 교정한 부분 (명시)**
- **"필드 로봇/로코모션" → "wheeled(휠) 로봇/locomotion"** : 슬라이드 제목이 "Wheeled Locomotion"이고 문맥(바퀴·RPM·평지·효율)이 일치. 전사 전반에 걸쳐 교정.
- 폐허→**페어(pair)**, 식단적→**직관적**, 각대→**(encoder) 값**, "고학년 대사기"→**공학용 계산기**, "트로퍼스/프로버스"→**True/False**, "r 곱하기 8"→**$r\dot\varphi$**(8≈$\dot\varphi$), "수율"→**휠**, "엘컴"(전사 불명)→문맥상 생략.
- 학회명 "아이크라/아이카/아이테라"(전사 불명) → **ICRA로 추정**하되 단정 않음.

**슬라이드와 전사가 어긋나거나 그림 의존으로 불확실한 부분**
- 차동 로봇에서 **left/right 바퀴의 $l$ 부호**($l=-b$ vs $+b$)는 $\alpha$를 $-90^\circ$로 두느냐 $+90^\circ$로 두느냐의 **표기 선택** 문제 — 슬라이드는 left $l=-b$, right $l=b$로 통일(p57–58). 어느 쪽을 잡아도 최종식 동일(전사 38:29).
- p50–51의 $^WR_R$ 행렬과 $l\sin\beta/l\cos\beta$ 항은 PNG 직접 확인으로 복원함(텍스트 추출 깨짐). 일반식 유도의 중간 대수(특히 $\hat\omega\,^I\mathbf r_{RW}$ 전개)는 슬라이드가 결과만 보여줘, 손유도 시 외적 전개를 직접 채워야 함.

**교수님이 "다음 시간에/복습"으로 남긴 것**
- 휠 로코모션을 더 일반적(Newton?) 기준으로 해석하는 내용은 **다음 시간 이어서**(전사 끝).
- **LMS 강의의 forward/inverse kinematics 문제 풀이 반드시 복습** (시험 변형 출제 예고).
- 다음 두 강의(6/11, 6/16): **torque, manipulator dynamics** → 기말 범위 포함.
