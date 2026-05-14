# ESP32 Quadruped RTOS & Network Study and Implementation Plan (Advanced Level)

Agent는 항상 이 글을 읽고 아래의 원칙에 따라 학습을 진행한다.

**문서 목적:** Good 설계와 Good Implement를 위한 RTOS/Network에 대한 학습 과정을 다루는 스터디 및 블로그 포스팅 계획서.

> **⚠️ 학습 목적 명시 (Learning Intent)**
> 이 패키지는 구현 워크스페이스가 아니다. 구체적인 구현에만 필요한 내용만 공부하는 것이 목적이 아니며, **"왜 이 선택이 옳은가, 혹은 왜 틀린가"를 스스로 판단할 수 있는 공학적 비판 능력**을 기르는 것이 핵심 목표이다.
> 
> 예를 들자면, ESP32에 직접 구현하기엔 과잉(Over-engineering)인 개념이더라도, **그것이 왜 과잉인지를 스스로 설명할 수 있는 수준**까지 이해하는 것을 요구한다.
> 모든 챕터는 "이것을 쓴다/안 쓴다"가 아닌, **"이것의 Trade-off를 나는 설명할 수 있는가"**를 기준으로 학습 완료 여부를 판단한다.

---

> **📝 학습 자료 작성 원칙 (Content Generation Policy)**
>
> 앞으로 생성되는 모든 문서와 포스팅은 아래의 11단계 교육학적 프레임워크(Pedagogical Framework)를 필수적으로 따른다.
>
> 0. 해당 기술의 명확한 개념과 정의를 설명한다.
>
> 1. The Deep Dive Explainer: "Break down [complex topic] like I'm 12, then gradually increase complexity over 5 levels until I reach expert understanding."
>
> 2. Mistake Prevention System: "List the 10 most common mistakes beginners make with [skill/topic]. For each, give me a simple check to avoid it."
>
> 3. Learning Path Architect: "Create a step-by-step roadmap to master [skill] in [timeframe]. Include milestones, resources, and weekly goals."
>
> 4. The Analogy Machine: "Explain [difficult concept] using 3 different analogies from [sports/cooking/movies]. Make it impossible to forget."
>
> 5. Practice Problem Generator: "Give me 5 progressively harder practice problems for [topic]. Include hints and detailed solutions."
>
> 6. Real-World Connector: "Show me 7 ways [concept I'm learning] applies to everyday situations. Use specific examples I can relate to."
>
> 7. Knowledge Gap Hunter: "Quiz me on [subject] with 10 questions. Based on my answers, identify exactly what I need to study next."
>
> 8. The Simplification Master: "Take this complex explanation [paste text] and rewrite it so a 10-year-old could understand it perfectly."
>
> 9. Memory Palace Builder: "Help me create a vivid story connecting these [facts/formulas/vocab words] so I never forget them."
>
> 10. Progress Accelerator: (학습 진척도 부스터 및 리마인더)
>
> 11. 작성 시 존대 대신 명확한 **-하다의 단언체**를 사용한다.

---

## 📚 Study & Implementation Workflow

기존의 단순 지식 전달에서 벗어나, 상기 **11가지 가이드라인을 전격 융합**하여 블로그 포스트를 완성한다. 각 단원은 필연적으로 **[Analogy(4번) & Deep Dive(1번)] $\rightarrow$ [Memory Palace(9번)] $\rightarrow$ [Hardware Verification(실측)] $\rightarrow$ [Mistake Prevention(2번) & Quiz(7번)]**의 구조를 띠게 된다.

**최종 판단 기준:** 각 챕터 학습 후, "이 기술의 설계 선택을 제3자에게 비판적으로 검토해달라는 요청을 받았을 때, 나는 Trade-off 근거를 들어 스스로 판단할 수 있는가?"

---

## Stage 1: RTOS 딥다이브 (로봇 생존의 심장과 물리적 정시성)

로봇의 다리가 꼬이지 않고 외부 충격에도 끄떡없는 제어 루프를 보장하기 위해, CPU 코어를 철저히 통제하여 오버헤드와 Jitter를 지배하는 과정.



### 0. Kernel Architecture & Determinism (RTOS 심층 원리)

* **[핵심 주제]** Hard/Firm/Soft Real-Time 시스템 분류, GPOS(CFS 스케줄러) vs RTOS(Priority-based Preemptive 스케줄러) 커널 레벨 오버헤드 비교.

* **[학습/비유]** GPOS는 '은행 번호표(CFS, 공평성)'이고 RTOS는 '응급실 환자 분류(Preemptive, 정시성)'이다. TCB 구조와 Context Switch의 PC/SP 레지스터 백업 원리를 5단계 딥다이브.

* **[실측 검증]** Bare-metal Super Loop와 FreeRTOS Task 간의 Dispatch Latency(디스패치 지연) 차이를 타이머로 벤치마킹하여 수치화한다.

* **[비판적 사고 훈련]** "RTOS를 쓰면 항상 더 빠른가?" → Context Switch 오버헤드와 TCB 관리 비용 때문에 단순 루프보다 느릴 수 있는 조건을 설명한다.

* **[오류/퀴즈]** Bare-metal에서 RTOS로 전환할 때 개발자가 저지르는 가장 흔한 설계 실수 체크리스트.



### 1-1. Multicore & Hard Real-Time Architecture (생존의 분리)

*   **[핵심 주제]** ESP32 듀얼 코어를 활용한 AMP(Asymmetric) 철학. Core 0(통신/센서)와 Core 1(동역학 제어)의 완벽한 분리 및 Preemptive 스케줄러의 하드웨어 커널 로직.

*   **[학습/비유]** GPOS(공평성) vs RTOS(정시성)의 차이를 '은행 번호표(CFS)'와 '응급실 환자 분류(Preemptive)'에 빗대어 설명하고, TCB 구조까지 5단계 딥다이브.

*   **[실측 검증]** Single 루프 설계와 Dual Core 분리 시 모터 제어 명령의 Jitter(가변폭)를 타이머로 벤치마킹.

*   **[비판적 사고 훈련]** "코어를 물리적으로 분리했는데 왜 여전히 병목이 생기는가?" → I/O 버스(I2C/SPI/GPIO) 는 코어와 무관하게 단일 물리 자원이므로, 코어 분리만으로는 해결되지 않는 병목의 존재를 설명한다.

*   **[오류/퀴즈]** 코어를 분리해 놓고도 I/O 병목에 걸려 같이 마비되는 치명적 실수(Mistake Prevention). 하드웨어 인터럽트 레벨 관련 퀴즈.



### 1-2. Deterministic Task Scheduling (무조건적인 심박동 보장)

*   **[핵심 주제]** `vTaskDelay` vs `vTaskDelayUntil`의 동작 원리와 Rate Monotonic Scheduling(우선순위 이론적 배정 기법).

*   **[학습/비유]** 드러머와 메트로놈(상대적 대기 분해), 릴레이 계주 바통 터치 등 3가지 비유(Analogy).

*   **[실측 검증]** Task 루프 내부에 고의적인 연산 지연(`delayMicroseconds`)을 주입하고, 누적 타임스탬프 편차(Drift)의 상쇄를 실측.

*   **[비판적 사고 훈련]** "`vTaskDelayUntil`을 쓰면 Jitter가 제거되는가?" → 아니다. 루프 내 Blocking I/O(I2C 트랜잭션 등)가 존재하는 한 Jitter는 여전히 발생한다. 스케줄러 수준의 해결책과 드라이버 수준의 해결책은 다른 계층의 문제이다.

*   **[오류/퀴즈]** Tick 변환(`pdMS_TO_TICKS`) 생략 누락 실수 예방 훈련. Jitter 누적 수학 모델링 연습문제.



### 1-3. Hardware Peripheral Blocking & Non-blocking I/O (진짜 병목의 위치)

*   **[핵심 주제]** I2C/SPI 통신의 **Blocking 특성**. CPU가 `Wire.write()`를 호출하는 순간 버스 전송이 완료될 때까지 아무것도 하지 못한다는 사실과, 이것이 실시간 제어 루프에 미치는 영향.

    > **이 챕터의 존재 이유:** vTaskDelayUntil로 20ms 루프를 설계했더라도, I2C로 PCA9685 서보 16채널 업데이트에 3~5ms의 Blocking이 발생하면, 설계한 결정론적 루프는 실제로 결정론적이지 않다. 이 사실을 모르는 채 "스케줄러만 통제하면 된다"고 생각하는 것이 임베디드 설계의 가장 흔한 오판이다.

*   **[학습/비유]** CPU가 I2C 버스를 '직접 운전하는 택시기사(Blocking)'와 'GPS 내비게이션에 경로를 맡기고 딴짓하는(DMA/Interrupt) 방식'으로 비유. I2C 버스 400kHz 클럭의 물리적 전송 시간을 직접 계산한다.

*   **[학습 심화]** 
    - **Polling 방식:** CPU가 버스 상태를 직접 확인하며 대기 (가장 단순, 가장 낭비적)
    - **Interrupt 방식:** 전송 완료 시 ISR 호출로 CPU 해방
    - **DMA 방식:** CPU 개입 없이 메모리-버퍼 간 직접 전송 (가장 효율적)

*   **[실측 검증]** `micros()`로 `Wire.endTransmission()` 전후를 측정하여 I2C Blocking 시간을 수치화한다. DMA 방식과의 CPU 점유율 차이를 비교한다.

*   **[비판적 사고 훈련]** "Arduino Wire 라이브러리는 왜 기본이 Blocking인가?" → 단순성과 이식성을 위한 설계 선택이다. 이 Trade-off를 이해하고, 비-Blocking 드라이버(ESP-IDF `i2c_master_transmit_async`)로의 전환이 언제 필요한지 판단하는 기준을 세운다.

*   **[오류/퀴즈]** Blocking I2C를 실시간 제어 루프 안에 그대로 두고 "vTaskDelayUntil을 썼으니 됐다"는 착각 사례 분석.



### 1-4. IPC & Data Tearing Defense (공용 메모리의 결함 방어)

*   **[핵심 주제]** 코어 간의 전역 변수 충돌(Data Tearing)과 Memory 경합을 막기 위한 Mutex 원자성 보장. Queue를 통한 데이터 이관.

*   **[학습/비유]** 교차로 신호등과 1칸짜리 공중화장실 열쇠 비유(Analogy). Priority Inversion(우선순위 역전)의 나비효과 설명.

*   **[실측 검증]** 통신 코어가 제어 목표값을 덮어쓰고 있는 찰나에 제어 코어가 데이터를 참조하여 로봇이 발작하는 현상 재현 및 Mutex 방패 구현.

*   **[비판적 사고 훈련]** "모든 공유 변수에 Mutex를 걸면 안전한가?" → 아니다. Mutex Lock을 잡고 있는 시간이 길어지면 반대로 정시성을 해친다. `volatile` 키워드만으로 해결되는 경우와 Mutex가 반드시 필요한 경우의 경계를 설명한다.

*   **[오류/퀴즈]** Mutex Lock을 잡은 채로 긴 연산을 돌리다 일어나는 데드락(Deadlock) 실수 체크리스트. 디버깅 회피 퀴즈.



### 1-5. Interrupt Deferred Processing & Hardware Watchdog (반사 신경과 불사조)

*   **[핵심 주제]** 충돌 등 하드웨어 인터럽트(ISR)의 커널 침범을 막는 `FromISR` 위임 패턴과, 프리즈 시 하드웨어 리셋을 유도하는 WDT(Watchdog Timer).

*   **[학습/비유]** 우편물 투척(ISR)과 수령인의 한가할 때 열어보기(Task) 스토리텔링(Memory Palace). 심박동이 멈추면 충격기를 작동하는 Watchdog의 개념.

*   **[실측 검증]** ISR 내부의 삼각함수 연산이 제어 주기를 박살내는 과정 측정, WDT 강제 타임아웃을 통한 시스템 자동 재부팅 벤치마킹.

*   **[오류/퀴즈]** ISR 함수에 `vTaskDelay`를 썼다 발생하는 패닉 에러. 인터럽트 내전성 관련 퀴즈.



### 1-6. Memory Management (Heap 단편화 방어)

* **[핵심 주제]** 임베디드 장치에서 동적 할당(malloc/new)의 위험성과 Heap Fragmentation(단편화).

* **[학습/비유]** 체스판 위의 색 분리처럼, 장기 동작 중 메모리가 점점 스위스 치즈처럼 구멍나는 과정을 시각화.

* **[실측 검증]** 메모리 크래시 현상을 시뮬레이션 해보고, 정적 할당(Static Allocation) 위주의 설계가 장기 신뢰성에 미치는 영향 학습.

* **[비판적 사고 훈련]** "FreeRTOS의 Heap_4/Heap_5 알고리즘은 왜 일반 malloc보다 낫다고 하는가?" → First-fit with coalescence 알고리즘의 동작 원리와 단편화 억제 메커니즘을 설명한다.

* **[오류/퀴즈]** 동적 할당과 정적 할당의 선택 기준 연습문제.

---

## Stage 2: Network 딥다이브 (통신 불확실성과 Fail-Safe)

아연실색할 무선 환경(끊김, 지연, 손실) 속에서도 로봇의 미쳐 날뜀(Runaway)을 원천 차단하는 가장 견고한 파이프라인.



### 2-1. Transport Layer Philosophy: UDP vs TCP (지연 vs 최신성 보장)

*   **[핵심 주제]** 데이터그램의 Fire-and-forget 특성과 로봇 텔레오퍼레이션 간의 본질적 적합성.

*   **[학습/비유]** TCP(등기 우편, 앞 우편이 늦으면 뒤 우편이 다 밀림 - HOL Blocking)와 UDP(최신 전단지 배포) 비유.

*   **[실측 검증]** PC $\rightarrow$ ESP32로 100Hz 명령 전송 시, TCP 환경에서 간헐적 지연이 발생하여 제어 지령이 덩어리(Burst)로 밀려 들어오는 파도 현상 비교 관찰.

*   **[비판적 사고 훈련]** "UDP를 쓰면 무조건 빠른가?" → 아니다. UDP도 ESP32의 lwIP 스택을 통해 처리되며, 패킷 처리 루틴 자체가 Core 0의 CPU 시간을 소비한다. 100Hz의 UDP 패킷은 Core 0에 어느 정도의 부하를 주는가?

*   **[오류/퀴즈]** 소켓 처리 로직이 Task를 Block시켜버리는 치명적 구조 오류(Mistake Prevention).



### 2-2. WiFi Stack Internals & CPU Overhead (보이지 않는 부하의 출처)

*   **[핵심 주제]** ESP32의 WiFi 드라이버는 단순한 라이브러리가 아니다. MAC 계층 처리, DHCP, lwIP TCP/IP 스택 전체가 **FreeRTOS Task로 Core 0에서 동작**하며, 애플리케이션 Task와 CPU 시간을 직접 경쟁한다.

    > **이 챕터의 존재 이유:** "UDP를 쓰고 Core 0에 통신 Task를 할당했으니 됐다"는 생각은 반쪽짜리다. WiFi 스택이 내부적으로 어떤 Task들을 스폰하고, 각각 얼마의 CPU를 쓰는지를 모르면, 예상치 못한 Jitter의 원인을 절대 찾을 수 없다.

*   **[학습 심화]**
    - `wifi_task` (MAC 계층 관리): 우선순위 23
    - `tiT` (TCP/IP lwIP 스택): 우선순위 18
    - `eventTask` (이벤트 루프): 우선순위 20
    - 위 Task들은 모두 `ESP_TASK_PRIO_*` 상수로 관리되며, 개발자의 Task보다 **우선순위가 높을 수 있다.**

*   **[학습/비유]** WiFi 스택을 'OS 위에서 돌아가는 또 다른 OS'로 비유. 네가 짠 Task는 WiFi가 처리를 마친 뒤에야 CPU를 얻을 수 있다는 사실을 교통 신호로 시각화.

*   **[실측 검증]** `uxTaskGetSystemState()`로 모든 FreeRTOS Task의 스택 사용량과 CPU 점유율을 덤프하여, WiFi 스택이 실제로 얼마의 CPU를 사용하는지 수치화한다. 100Hz vs 50Hz 수신 시의 CPU 점유율 차이를 비교한다.

*   **[비판적 사고 훈련]** "WiFi Task 우선순위가 내 제어 Task보다 높다면 어떻게 해야 하는가?" → `CONFIG_ESP32_WIFI_TASK_PINNED_TO_CORE_0` 설정으로 WiFi를 Core 0에 고정하여 Core 1의 제어 Task를 물리적으로 격리하는 전략과 그 한계를 설명한다.

*   **[오류/퀴즈]** WiFi Task 우선순위와 애플리케이션 Task 우선순위의 충돌로 인한 Starvation 사례 분석.



### 2-3. Serialization & Struct Alignment (이기종 간의 언어 통일)

*   **[핵심 주제]** PC(x86/ROS2)와 ESP32(Xtensa) 간의 프로세서 아키텍처 차이(Endianness)와 구조체 패딩(Padding).

*   **[학습/비유]** 서로 다른 언어를 구사하는 외교관의 통역(Endianness)과 울퉁불퉁한 짐을 빈틈없이 채워 넣는 테트리스 포장(Padding) 비유.

*   **[실측 검증]** `#pragma pack(1)` 누락 시 C++에서 쓰레기 값이 수신되는 이유를 바이트 수준(Pointer Casting)에서 덤프 및 디코딩하여 증명.

*   **[오류/퀴즈]** `float` 데이터를 배열로 보낼 때 네트워크 바이트 오더(Big Endian) 통일 누락 실수 체크리스트.



### 2-4. Network Disconnection FSM & Non-blocking Reconnect (통신 두절 생존기)

*   **[핵심 주제]** 무선 통신이 완전히 소실되었을 때 제어를 멈추지 않고 Safe Mode(안전 자세)로 유도하는 상태 머신과 백그라운드 재연결 알고리즘.

*   **[학습/비유]** 절벽 등반 중 한 손으로 끊어진 동아줄을 매듭지으며 다른 손으론 절벽을 버텨내는 끝없는 생명력 비유(Memory Palace).

*   **[실측 검증]** 와이파이 전원 차단 시 즉각적으로 Target 제어값을 0 처리(정지)하며 Core 0 모듈만 백그라운드 재연결(Non-blocking Begin)을 무한 시도하는 로직 구현.

*   **[오류/퀴즈]** `WiFi.begin()` 함수의 악명 높은 Blocking 특성을 무시했다가 로봇 시스템 전체가 먹통이 되는 실수 방지. 네트워크 회복탄력성 연습문제.



### 2-5. Latency Jitter & Time Synchronization (원격 제어의 물리적 한계선)

*   **[핵심 주제]** Network RTT(Round-Trip Time) 측정을 통한 지연(Jitter) 모니터링 및 패킷 유실률 감지. 초과 시 즉각 Fail-Safe 발동.

*   **[학습/비유]** 심해 잠수부(로봇)와 지상 통제선(PC) 간의 산소 핑(Ping) 테스트 비유(Real-World Connector).

*   **[실측 검증]** 데이터 패킷 내에 시퀀스(Sequence Number)와 타임스탬프를 심고 왕복시켜 RTT 50ms 초과 시 강제 연결 해제 로직 작동 테스트.

*   **[오류/퀴즈]** PC와 ESP32 간의 기기별 시계(RTC) 오차를 고려하지 않는 실수. 지연 모델 역기구학 오차 문제 계산(Practice Generator).

---

## Stage 3: Architecture Philosophy (설계 판단력 확장)

단순히 코드를 짜는 능력을 넘어, **하나의 아키텍처 패턴이 왜 특정 환경에 적합하고, 왜 다른 환경에서는 독이 되는지**를 판단하는 눈을 기른다.



### 3-1. Message Passing Paradigms: Queue vs Task Notification vs Pub-Sub (아키텍처 라이브러리)

*   **[핵심 주제]** 모듈 간 통신을 위한 세 가지 패러다임의 내부 동작과 오버헤드의 차이. "언제 무엇을 쓰는가"의 판단 기준.

    > **이 챕터의 존재 이유:** Pub-Sub 패턴을 모르면 "왜 ROS 2가 그것을 쓰는가"를 모른다. 하지만 Pub-Sub 패턴만 알면 "왜 ESP32에서 그것을 쓰면 안 되는가"를 모른다. 이 챕터의 목표는 두 가지를 모두 설명할 수 있는 것이다.

*   **[학습 심화]**

    | 패러다임 | 오버헤드 | 적합 환경 | 핵심 비용 |
    |---|---|---|---|
    | `Task Notification` | **극소** | MCU 내부, 단순 이벤트 신호 | TCB 플래그 직접 조작 |
    | `Queue` | 소 | MCU 내부, 데이터 전달 필요 시 | 메모리 복사 비용 |
    | **Pub-Sub Broker** | **중~대** | ROS 2, Linux, 마이크로서비스 | 구독자 목록 순회 + 동적 디스패치 |

*   **[학습/비유]** Task Notification은 '어깨를 탁 치는 직접 신호(1:1 빠름)', Queue는 '공용 우편함(데이터 보존)', Pub-Sub은 '방송국 시스템(1:N 유연성, 그러나 스튜디오 운영 비용 발생)'으로 비유.

*   **[심층 분석: Pub-Sub의 실제 비용]**
    - 구독자 목록을 **동적으로 관리** → Heap 할당 발생 (1-6챕터에서 금지한 바로 그것)
    - 메시지 발행마다 구독자 수만큼 `xQueueSend` 루프 실행 → Task 스케줄러 경합
    - 브로커 Task 자체가 CPU 시간 일부를 항상 소비
    - **결론:** ESP32 내부에서 이것을 구현하면, 제어 루프가 스스로 만든 오버헤드에 의해 Jitter가 늘어난다.

*   **[심층 분석: ROS 2가 Pub-Sub을 쓰는 이유]**
    - Linux 환경에서는 CPU 코어가 여러 개이고 메모리가 충분하다.
    - 노드(프로세스) 간의 인터페이스 명세가 코드 수정 없이 유연하게 확장되어야 한다.
    - 브로커 오버헤드 대비 유지보수성과 확장성의 Trade-off가 압도적으로 유리하다.

*   **[비판적 사고 훈련]** "그렇다면 ESP32에서 ROS 2의 철학(약결합)을 구현하고 싶다면 어떻게 하는가?" → Pub-Sub 브로커 없이도 **공유 구조체 + Mutex + Task Notification**의 조합으로 동일한 모듈 독립성을 달성할 수 있다. 단, 이것은 컴파일 타임에 고정된 정적 약결합이다.

*   **[오류/퀴즈]** "좋은 아키텍처 패턴은 항상 더 좋은 코드를 만드는가?" → 아니다. 아키텍처 패턴의 적합성은 항상 실행 환경의 제약 조건에 종속된다. 연습문제: 주어진 시스템 사양(RAM, CPU 속도, Task 수)에서 각 패러다임의 적합성을 판단하라.



### 3-2. Premature Optimization & Over-engineering (판단 실패의 해부학)

*   **[핵심 주제]** "좋은 설계"와 "복잡한 설계"는 다르다. 자원이 제한된 환경에서 고수준 패턴을 그대로 이식하려는 욕구가 어떻게 치명적인 역효과를 낳는가.

    > 이 챕터에서 다루는 핵심 질문: **"나는 지금 문제를 해결하고 있는가, 아니면 패턴을 이식하고 있는가?"**

*   **[학습/비유]** 도심 배달을 위해 18륜 트럭을 구입하는 것(Pub-Sub on ESP32). 화물 적재 능력은 뛰어나지만, 골목길을 못 다닌다.

*   **[학습 심화]** Donald Knuth의 "Premature Optimization is the root of all evil"의 진짜 의미. 코드가 복잡해질수록 디버깅 난이도는 지수적으로 증가하며, MCU 환경에서 이는 직접적인 신뢰성 하락으로 이어진다.

*   **[비판적 사고 훈련]** 주어진 시스템 요구사항에서 "이 설계가 Over-engineering인가 아닌가"를 판단하는 3가지 질문:
    1. 이 패턴이 해결하는 문제가 현재 시스템에서 실제로 발생하는가?
    2. 이 패턴의 오버헤드가 시스템의 핵심 제약(실시간성, 메모리)에 영향을 주는가?
    3. 더 단순한 방법으로 동일한 목표를 달성할 수 있는가?

*   **[오류/퀴즈]** 실제 임베디드 시스템 사례에서 Over-engineering이 야기한 실패 사례 분석 및 리팩토링 연습.

---

## 📝 Blog Posting 마일스톤

수행한 스터디와 논의, 코드 레이어 검증 결과를 바탕으로 `_posts` 에 실질적인 가치를 지닌 기술 블로그 포스트를 지속 발행한다.

* **Focus:** AI가 짜준 코드를 맹목적으로 구동하는 데 그치지 않고, "어떤 문제를 마주했고", "왜 특정 기술을 선택했는지(Trade-off 분석)", "어떻게 스스로 검증했는지"에 초점을 맞춰 작성한다.

* **최종 목표:** 단순한 튜토리ㄴ얼을 벗어나, **로봇 제어 공학과 컴퓨터 과학 전공 지식**이 충돌하는 지점을 파고들며 **11개의 교육학적 장치**를 통해 압도적인 품질의 기술 블로그를 연재한다. 궁극적으로는, **이 계획서에 담긴 모든 Trade-off를 제3자에게 비판적으로 설명하고 방어할 수 있는 수준**에 도달하는 것이 학습 완료의 기준이다.