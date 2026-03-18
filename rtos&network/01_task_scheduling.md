# Stage 1-1: Task Scheduling & Timing (심장 박동 제어)

로봇(장치) 제어의 핵심은 **"정해진 시간(주기)마다 정확하게 동작하는 것(Determinism)"**입니다. 초당 50번(50Hz, 20ms 주기) 모터에 명령을 내려야 하는 시스템에서, 연산 시간에 따라 주기가 1ms씩 계속 뒤로 밀린다면 어떻게 될까요? 결국 글로벌 타임라인이 붕괴되어 로봇은 의도한 궤적을 이탈하게 됩니다.

RTOS에서 Task의 주기를 제어하는 대표적인 두 가지 함수 `vTaskDelay`와 `vTaskDelayUntil`을 전격 비교합니다.

---

## 🕒 1. vTaskDelay의 맹점 (상대적 지연)

`vTaskDelay(ticks)` 함수는 **현재 이 함수가 호출된 시점으로부터** 지정된 틱(Tick)만큼 대기상태(Blocked)로 들어갑니다.

### ❌ 문제점: "내가 늦게 끝나면 다음 일도 늦게 시작한다"
만약 20ms 주기로 동작해야 하는 `Control_Task`가 있다고 가정해 봅시다.

1. Task가 깨어납니다. (T = 0ms)
2. 복잡한 역운동학 연산을 수행합니다. 이 연산에 평소에는 2ms가 걸리다가, 특정 복잡한 조건에서는 5ms가 걸렸다고 칩시다.
3. 연산이 끝나고 `vTaskDelay(20)`을 호출합니다. (T = 5ms 시점)
4. 다음 Task는 언제 깨어날까요? 바로 T = 25ms (5ms + 20ms) 시점입니다.
5. 원래 의도한 두 번째 주기는 T = 20ms 시점이었지만, 연산 시간 5ms만큼 영구적으로 뒤로 밀려버렸습니다. (누적 오차 발생)

```cpp
void Control_Task(void *pvParameters) {
    while (true) {
        // [연산] 때때로 1ms ~ 5ms 등 소요 시간이 변동됨 (Jitter)
        calculate_kinematics(); 
        
        // [대기] "지금부터" 20ms 쉴게! -> 연산 시간만큼 다음 주기가 누적해서 밀림
        vTaskDelay(20 / portTICK_PERIOD_MS);
    }
}
```

---

## ✅ 2. vTaskDelayUntil의 위력 (절대적 지연)

`vTaskDelayUntil(&PreviousWakeTime, ticks)` 함수는 **이전 기상 시간(PreviousWakeTime)을 기억해 두고, 그 시간을 기준으로** 정확히 틱(Tick)만큼만 자고 일어납니다.

### 💡 해결책: "연산 시간이 얼마든, 약속한 시간에 무조건 기상한다"
똑같은 20ms 주기 `Control_Task`를 살펴봅시다.

1. Task가 처음 깨어난 시간(`PreviousWakeTime`)을 저장합니다. (T = 0ms)
2. 복잡한 역운동학 연산을 수행하여 5ms가 소요되었습니다.
3. 연산이 끝나고 `vTaskDelayUntil(&PreviousWakeTime, 20)`을 호출합니다.
4. RTOS 커널이 계산합니다: *"이전 기상 시간(T=0)에서 20ms를 더한 T=20 시점까지 몇 ms 남았지? 알겠어, 연산에 5ms 썼으니 남은 15ms만 자고 일어나!"*
5. 결과적으로 다음 Task는 정확히 T = 20ms 시점에 칼같이 깨어납니다. 주기가 완벽하게 지켜집니다.

```cpp
void Control_Task(void *pvParameters) {
    // 최초 기상 시간을 기준점으로 캡처
    TickType_t xLastWakeTime = xTaskGetTickCount();
    const TickType_t xFrequency = 20 / portTICK_PERIOD_MS; // 20ms (50Hz)

    while (true) {
        // [연산] 소요 시간이 변동되어도 상관없음
        calculate_kinematics(); 

        // [대기] "기준점(xLastWakeTime)으로부터 정확히 20ms 뒤에 깨워줘!"
        // xLastWakeTime은 함수 내부에서 자동으로 다음 기준점(T+20)으로 갱신됨
        vTaskDelayUntil(&xLastWakeTime, xFrequency);
    }
}
```

---

## 🛠 3. Action Item (우리가 실험할 내용)

이론을 아는 것과 눈으로 확인하는 것은 천지 차이입니다. 다음 스텝에서는 ESP32(혹은 에뮬레이터)를 활용해 직접 두 함수의 차이를 코드로 박아넣고 시리얼 모니터로 관찰할 것입니다.

**[실험 시나리오]**
1. 2개의 Task(`Task_Delay`, `Task_DelayUntil`)를 나란히 생성합니다.
2. 각 Task는 목표 주기를 `100ms(10Hz)`로 설정합니다.
3. 루프 내부에 `delayMicroseconds(random(1000, 50000))` (1ms ~ 50ms)의 고의적인 랜덤 렉(부하)을 주입합니다.
4. 각 Task가 10번째, 100번째 실행될 때의 **현재 시스템 타임스탬프(`millis()`)**를 출력해 봅니다.
5. **결과 예측:** 
   - `Task_Delay`는 100번째 실행 시 `10,000ms`가 한참 지난 누적 오차가 발생한 시간이 찍힙니다.
   - `Task_DelayUntil`은 렉이 걸렸음에도 칼같이 `10,000ms` 근방(정확한 주기)이 찍히는 것을 직접 눈으로 확인합니다.

> 📝 **준비되셨다면, 이 "실험 시나리오"를 ESP32 C++ 코드로 작성해 드리고 이를 직접 돌려보는 단계를 밟아볼까요?**
