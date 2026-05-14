---
title: "1. Task Scheduling & Timing"
description: "로봇 제어에서 Jitter를 통제하기 위한 vTaskDelay와 vTaskDelayUntil의 분석 및 누적 오차 검증"
author: KEYBEOMM
date: 2026-03-20 03:00:00 +0900
categories: [Embedded Systems, RTOS]
tags: [rtos, freertos, esp32, scheduling, timing]
---

로봇 하드웨어 제어의 핵심은 **'정확한 주기의 반복'**입니다. 지연 시간의 편차인 Jitter를 통제하지 못하면 제어 궤적은 반드시 무너집니다. 
본 포스팅에서는 직관적인 비유와 실제 커널 동작 레벨의 분석을 결합하여, 왜 로봇 제어 루프에서 특정 Timing API를 선택해야 하는지 탐구합니다.

---

## 1. Intuitive Analogy:

로봇의 다리를 10ms(100Hz)마다 제어해야 하는 Task를 밴드의 **드러머**라고 가정해 보겠습니다. 이 드러머는 1초마다 정확히 드럼을 쳐야 합니다.

### 상대적 연기 (vTaskDelay)
드러머가 스스로 마음속으로 초를 세고 치는 방식입니다.
"드럼을 치고 나서, **지금부터** 1초 뒤에 쳐야지"라고 생각합니다. 만약 드럼채를 떨어뜨려 줍느라 0.1초를 낭비했다면? 다음 드럼은 1.1초 뒤에 치게 됩니다. 이 0.1초의 지연은 다음 주기, 다다음 주기에도 계속 누적(Drift)되어 결국 밴드의 박자는 완전히 무너집니다.

### 절대적 연기 (vTaskDelayUntil)
드러머가 마음속으로 초를 세지 않고 눈앞의 **메트로놈(절대 시간계)**을 보고 치는 방식입니다.
드럼채를 줍느라 0.1초를 낭비했더라도 드러머는 손목시계를 보고 "다음 정각(1초)에 쳐야 하니까, 이번에는 0.9초만 기다려야겠다"라고 보상(Compensation) 동작을 수행합니다. 누적 오차가 전혀 발생하지 않습니다.

---

## 2. Professional CS Depth: vTaskDelay vs vTaskDelayUntil

FreeRTOS 환경에서 Task를 잠재우는 대기(Block) 상태 전환 메커니즘을 커널 레벨에서 분석해 보겠습니다.

### ① vTaskDelay (상대 시간 대기)

`vTaskDelay(ticks)`는 **API가 호출된 시점**을 기준으로 전달된 매개변수 값만큼 Task를 Blocked 리스트에 둡니다.

```c
void vTask1(void *pvParameters) {
    while(1) {
        // [1] 센서 읽기 & 제어 연산 수행
        doHeavyCalculation(); 
        
        // [2] 함수가 '호출된 시점'부터 10ms 대기
        vTaskDelay(pdMS_TO_TICKS(10)); 
    }
}
```

* **동작 원리와 한계점:**
  `doHeavyCalculation()` 연산에 **2ms**가 소요되었다면, 한 루프를 도는 전체 시간은 **2ms(연산) + 10ms(대기) = 12ms** 가 됩니다.
  만약 인터럽트(ISR)나 더 높은 우선순위 Task가 선점(Preemption)하여 연산 시간이 4ms로 늘어났다면, 주기는 14ms가 됩니다. 
  즉, Task의 실행 주기(Frequency)가 이전 작업의 수행 시간이나 OS 스케줄링 간섭 정도에 따라 계속해서 변하게(Drift) 되며, Hard Real-Time 에서는 사용할 수 없는 형태의 스케줄링입니다.

### ② vTaskDelayUntil (절대 시간 대기)

`vTaskDelayUntil(&xLastWakeTime, ticks)`는 변수에 저장된 **Task가 마지막으로 깨어났던 절대 시점(`xLastWakeTime`)**을 기준으로 대기합니다. 즉, 이전에 코어가 어떤 작업으로 지연을 겪었든 간에 **일정한 주파수(Hz)**를 칼같이 맞춰줍니다.

```c
void vControlTask(void *pvParameters) {
    // 1. Task가 최초로 시작될 때의 절대 시각(Tick Count)을 기록
    TickType_t xLastWakeTime = xTaskGetTickCount();
    const TickType_t xFrequency = pdMS_TO_TICKS(10); // 10ms 주기 (100Hz)

    while(1) {
        // [1] 제어 연산 (시간 변동 발생)
        doHeavyCalculation();

        // [2] '이전에 깨어난 시점'을 기준으로 정확히 10ms 뒤에 깨어나도록 예약
        // 스케줄러가 알아서 xLastWakeTime 값을 다음 목표 시각으로 자동 갱신함
        vTaskDelayUntil(&xLastWakeTime, xFrequency);
    }
}
```

* **동작 원리와 장점:**
  FreeRTOS 커널은 이전에 깨어났던 시각과 목표 주기(xFrequency)를 이용해 잠들어야 할 정확한 타이밍을 역산합니다. 즉, `doHeavyCalculation()` 연산에 소요된 시간을 자동으로 차감하여 대기 시간을 조정합니다.
  결과적으로 **제어 루프의 실행 주기는 언제나 정확히 10ms(100Hz)로 고정(Determinism) 보장**을 받습니다.

---

## 3. Action Item & Jitter 검증

이론적 원리를 하드웨어 코드로 직접 검증하기 위해 빈 Task 2개를 생성하고, 루프 내부에 고의적인 부하(`delayMicroseconds`)를 주입한 뒤 두 API의 Jitter를 시리얼 타임스탬프로 실측합니다.

### 검증 코드 (ESP32 / FreeRTOS)

```cpp
#include <Arduino.h>

// 1. 상대적 대기 테스트용 Task
void vTaskRelative(void *pvParam) {
    while(1) {
        // [인위적 부하] 1ms ~ 3ms 사이의 무작위 딜레이 시간(Block이 아닌 CPU 독점 대기) 주입
        delayMicroseconds(random(1000, 3000)); 
        
        Serial.print("[vTaskDelay] Timestamp: ");
        Serial.println(millis());
        
        // 함수 호출 시점부터 10ms 대기
        vTaskDelay(pdMS_TO_TICKS(10)); 
    }
}

// 2. 절대적 대기 테스트용 Task
void vTaskAbsolute(void *pvParam) {
    TickType_t xLastWakeTime = xTaskGetTickCount();
    while(1) {
        // [인위적 부하] 1ms ~ 3ms 사이의 무작위 딜레이 시간 주입
        delayMicroseconds(random(1000, 3000));
        
        Serial.print("[vTaskDelayUntil] Timestamp: ");
        Serial.println(millis());
        
        // 마지막 시점을 기준으로 강제 간격(10ms) 유지
        vTaskDelayUntil(&xLastWakeTime, pdMS_TO_TICKS(10)); 
    }
}

void setup() {
    Serial.begin(115200);
    // 각각의 테스트 Task 생성
    xTaskCreate(vTaskRelative, "Task_Rel", 2048, NULL, 1, NULL);
    xTaskCreate(vTaskAbsolute, "Task_Abs", 2048, NULL, 1, NULL);
}

void loop() {}
```

### 타임스탬프 결과 분석
* **`vTaskDelay` 출력 형태:** 주기가 12ms, 13ms, 11ms 씩 제각각 걸리게 되어 타임스탬프가 `12, 25, 36, 49 ...` 식으로 간격이 벌어집니다. 앞선 연산의 지연 시간이 다음 주기에 영구적인 누적 오차 생성을 촉발합니다.
* **`vTaskDelayUntil` 출력 형태:** 타임스탬프가 `10, 20, 30, 40 ...` 으로 찍힙니다. 루프 내부에 있는 막대한 연산 지연의 존재를 스케줄러가 정확하게 상쇄 보정하여 10ms의 절대적인 심박동(Heartbeat)을 지켜냅니다.

> **결론**
> 모터, 센서 등 현실 세계의 동역학 모델과 직접적으로 소통하는 Hard Real-Time 제어기에서는 반드시 **`vTaskDelayUntil`**을 활용하여 뼈대(Control Loop)를 구축해야 합니다.
