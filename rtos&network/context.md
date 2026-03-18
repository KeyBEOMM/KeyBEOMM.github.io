# ESP32 Quadruped RTOS & Network Study and Implementation Plan (Advanced Level)

Agent는 항상 이 글을 읽고 아래의 원칙에 따라 학습을 진행한다.

**문서 목적:** 직관적인 비유와 심도 있는 전공 전산학 지식을 완벽히 결합하여, 실무에서 마주하는 하드웨어 아키텍처의 한계와 RTOS/Network의 딥다이브(Deep-Dive) 과정을 다루는 스터디 및 블로그 포스팅 계획서.

> **📝 학습 자료 작성 원칙 (Content Generation Policy)**
> 앞으로 생성되는 모든 문서와 포스팅은 아래의 3단계 흐름(Hybrid Approach)을 필수적으로 따릅니다.
> 1. **Intuitive Analogy (직관적 비유):**  복잡한 기술이 "왜 필요한지" 철학적 배경을 이해시킨다.
> 2. **Professional CS Depth (전공자 수준의 딥다이브):** 비유에 머물지 않고 실제 커널 아키텍처(TCB, Context Switch Overhead), 스케줄링 알고리즘(CFS vs Preemptive), 메모리 정렬 등 하드코어한 학부 전공 용어와 동작 원리를 파헤친다. 전공 용어의 경우 정확한 정의와 개념을 설명한다.


> **Objective:** 
> - **Deep RTOS:** OS 스케줄러 알고리즘, 하드웨어적 Context Switch 오버헤드, Mutex/Semaphore의 커널 레벨 지식을 제어 코드와 결합하여 검증.
> - **Deep Network:** 네트워크 계층 모델 최하단의 전송 지연(RTT), 직렬화 패딩오차(Struct Padding), 비동기 소켓 논블로킹(Non-blocking I/O) 구조 분석.
> - **Outcome:** 엔지니어링의 Trade-off를 '직관적인 비유'와 '전문적인 수치'로 완벽히 엮어낸 최고 품질의 기술 블로그 발행.

---

## 📚 Study & Implementation Workflow
**[비유와 전공 지식을 결합한 CS 심층 토론]** $\rightarrow$ **[하드웨어 레벨의 코드 검증 및 지표(Metric) 측정]** $\rightarrow$ **[트러블슈팅 포스팅]**

## Stage 1: RTOS Deep-Dive (Kernel & Synchronization)
ESP32/FreeRTOS의 커널 동작 원리 및 멀티코어 환경의 제어 아키텍처.

### 0. Kernel Architecture & Determinism (RTOS 심층 원리)
* **[Topic]** Hard/Firm/Soft Real-Time 시스템 분류, GPOS(CFS 스케줄러) vs RTOS(Priority-based Preemptive 스케줄러) 커널 레벨 오버헤드 비교.
* **[Action Item]** 하드웨어 레지스터 관점의 Context Switch(PC, SP 상태 백업) 프로세스 및 TCB의 구조 이해. Bare-metal(Super Loop) 환경 대비 디스패치 지연(Dispatch Latency)의 한계점 파악.
* **[Post-Goal]** "펌웨어 설계 아키텍처: Bare-metal Super Loop 시스템에서 Preemptive 멀티 스레딩으로의 패러다임 전환과 오버헤드 정량 분석" 포스트 발행.

### 1. Task Scheduling & Timing (심장 박동 제어)
로봇 제어의 핵심은 '정확한 주기의 반복'이다. 지연(Jitter)을 통제하지 못하면 궤적은 반드시 무너진다.
* **[Topic]** `vTaskDelay` vs `vTaskDelayUntil`의 동작 원리와 로봇 제어 관점에서의 차이 분석
* **[Action Item]** 빈 Task를 생성하고 루프 내부에 고의적인 부하(`delayMicroseconds`)를 주입한 뒤, 두 API의 시리얼 타임스탬프 누적 오차를 직접 비교 및 탐구.
* **[Post-Goal]** "왜 로봇 제어에서는 vTaskDelay가 아닌 vTaskDelayUntil을 써야 하는가?"에 대한 실험적 증명 작성.

### 2. IPC (Inter-Process Communication)와 Data Tearing
Core 0(통신)와 Core 1(제어)의 메모리 경합을 방어한다.
* **[Topic]** Data Tearing 현상에 대한 이해와 Mutex(상호 배제)의 동작 원리 파악.
* **[Action Item]** Core 1이 제어 변수를 읽는 도중 Core 0가 덮어쓰는 상황을 이해하고, Lock을 쥐고 있는 시간을 '마이크로초' 단위로 억제해야 하는 이유(Priority Inversion 방지) 조사.
* **[Post-Goal]** "멀티코어 환경의 생존 법칙: Mutex와 Priority Inversion" 요약 및 기술 블로그 포스팅.

### 3. Queue & Interrupt Handling (ISR과 Task의 협력)
* **[Topic]** 하드웨어 인터럽트(ISR)와 비동기 이벤트의 안전한 처리: Deferred Interrupt Processing.
* **[Action Item]** 타이머 인터럽트 내에서 무거운 연산을 하지 않고, Queue를 활용해 Event Task로 처리 로직을 위임(`FromISR` API 활용)하는 패턴 구현.
* **[Post-Goal]** "RTOS 인터럽트 디자인 패턴: ISR 수행 시간 최소화와 Task 위임" 포스트 작성.

### 4. Memory Management (Heap 단편화 방어)
* **[Topic]** 임베디드 장치에서 동적 할당(malloc/new)의 위험성과 Heap Fragmentation(단편화).
* **[Action Item]** 메모리 크래시 현상을 시뮬레이션 해보고, 정적 할당(Static Allocation) 위주의 설계가 장기 신뢰성에 미치는 영향 학습.
* **[Post-Goal]** "안정적인 로봇 제어기를 위한 고집: 동적 메모리 할당 제거하기" 블로그 포스팅.

---

## Stage 2: Communication Pipeline (네트워크 불확실성 방어)
PC(ROS 2)와 ESP32 간의 무선 통신 생태계와 패킷의 물리적 한계를 이해한다.

### 1. 프로토콜 선정: TCP vs UDP
* **[Topic]** 로봇 통신에 있어 과거의 정상 명령을 늦게라도 보장받는 것(TCP)과 최신 명령을 유실 감수하고 꽂아넣는 것(UDP)의 철학적 및 실질적 차이 고찰.
* **[Action Item]** UDP 소켓의 'Fire-and-forget' 특성을 학습하고, 패킷 유실을 소프트웨어적으로 억제·방어하기 위해 구조체에 `timestamp`를 포함하는 설계의 당위성을 검증.
* **[Post-Goal]** "제어기 통신에서 TCP 대신 UDP를 선택하는 엔지니어링적 이유" 분석 포스트 작성.

### 2. Struct Packing & Endianness (메모리 정렬 오차)
* **[Topic]** PC(x86)와 ESP32(Xtensa) 간의 패딩(Padding) 방식 파악 및 직렬화.
* **[Action Item]** C++ 구조체 전송 시 발생하는 메모리 정렬 오차와 쓰레기 값을 분석하고, `#pragma pack(1)` 또는 `__attribute__((packed))`의 의미를 학습하여 더미 데이터로 통신 검증.
* **[Post-Goal]** "이기종 디바이스 간 통신의 함정: Struct Packing의 필요성" 이슈 및 해결 과정 정리.

### 3. Connection Stability & Non-blocking Reconnect
* **[Topic]** 무선 통신(Wi-Fi) 끊김 현상 대처와 제어 루프를 멈추지 않는(Non-blocking) 재연결 아키텍처 파악.
* **[Action Item]** 통신이 끊겼을 때 제어 루프(Core 1)가 멈춤 없이 안전 모드로 전환하여 구동을 지속하고, Core 0가 백그라운드 재연결을 시도하도록 구현 및 테스트.
* **[Post-Goal]** "무선 제어 로봇의 생명단축 방지: Non-blocking Wi-Fi 재연결 로직 구현" 블로그 포스팅.

### 4. Latency & Jitter Measurement (Ping-Pong 테스트와 안전망)
* **[Topic]** 네트워크 왕복 지연 시간(RTT, Round-Trip Time) 측정 및 통신 지연(Jitter) 모니터링 고찰.
* **[Action Item]** PC-ESP32 간 타임스탬프를 통신 패킷에 담아 왕복시켜 라우터 Latency를 측정하고, 지연 한계 초과 시 로봇을 안전 상태로 강제 전환하는 Fail-Safe 로직 구현.
* **[Post-Goal]** "눈에 보이지 않는 통신 지연(Latency)의 측정과 제어기 Fail-Safe 시스템" 포스트 작성.

---

## 🔜 Stage 3: (Separated) 
* **안내:** 기존 기획되었던 `Bottom-up Mocking & Verification Pipeline` (기구학 결합 및 단일 다리 하드웨어 테스트 등 통합 검증 단계)는 **해당 폴더에서 진행하지 않으며, 다른 패키지 공간에서 독립적으로 별도 진행**합니다.

---

## 📝 Blog Posting 마일스톤
수행한 스터디와 논의, 코드 레이어 검증 결과를 바탕으로 `_posts` 에 실질적인 가치를 지닌 기술 블로그 포스트를 지속 발행합니다.
* **Focus:** AI가 짜준 코드를 맹목적으로 구동하는 데 그치지 않고, "어떤 문제를 마주했고", "왜 특정 기술을 선택했는지(Trade-off 분석)", "어떻게 스스로 검증했는지"에 초점을 맞춰 작성합니다.
