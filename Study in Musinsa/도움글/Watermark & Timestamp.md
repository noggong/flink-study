## **✅ Flink의 타임스탬프(Timestamp)와 워터마크(Watermark) 개념 & 활용 예제**

> **🔥 Flink에서 스트리밍 데이터를 시간 기반으로 처리할 때 핵심이 되는 개념**  
> **👉 이벤트 기반 타임스탬프 + 워터마크를 활용하면 지연 데이터(Out-of-Order Events)도 정확하게 처리 가능**

---

# **🚀 1️⃣ 타임스탬프(Timestamp)란?**

### **📌 스트리밍 데이터에서 시간 개념을 부여하는 방식**

Flink에서는 **스트리밍 데이터를 시간 기반으로 분석할 때, 사용할 시간을 선택해야 함**.

### **✅ Flink의 시간 유형 3가지**

| 시간 유형               | 설명                                                        |
| ------------------- | --------------------------------------------------------- |
| **Processing Time** | 이벤트를 처리하는 **현재 시스템 시간** 기준 (`System.currentTimeMillis()`) |
| **Event Time**      | 이벤트가 실제 **발생한 시간** (`타임스탬프를 데이터에서 추출`)                    |
| **Ingestion Time**  | 데이터가 **Flink에 유입된 시간** 기준                                 |

✅ **대부분의 스트리밍 분석에서는 `Event Time`을 사용**  
✅ **Flink는 이벤트 시간(`Timestamp`)을 직접 설정해야 함**

---

# **🚀 2️⃣ 워터마크(Watermark)란?**

### **📌 데이터가 지연될 수 있는 스트리밍 환경에서 정확한 시간 연산을 보장하는 기법**

- **스트리밍 데이터는 네트워크 지연, 장애 등으로 인해 순서가 뒤섞일 수 있음**
- **Flink는 `Watermark`를 사용하여 데이터가 늦게 도착했더라도 정확한 시간 기반 연산을 수행 가능**

✅ **즉, `Watermark`는 "지금까지 도착한 데이터에서 믿을 수 있는 가장 늦은 시간"을 의미**  
✅ **Watermark가 특정 시점을 넘어서면 그보다 더 이전의 데이터는 도착하지 않는다고 가정**

---

# **🚀 3️⃣ Flink에서 `Timestamp` & `Watermark` 설정 방법**

## **✅ 예제 1: 타임스탬프를 직접 설정하는 방법**

kotlin

복사편집

`val env = StreamExecutionEnvironment.getExecutionEnvironment()  // ✅ 데이터 스트림 생성 (예: Kafka에서 수집) val inputStream: DataStream<MyEvent> = env.addSource(KafkaSource())   // ✅ 타임스탬프 추출 및 워터마크 생성 val streamWithTimestamps = inputStream     .assignTimestampsAndWatermarks(         WatermarkStrategy.forBoundedOutOfOrderness<MyEvent>(Duration.ofSeconds(5))             .withTimestampAssigner { event, _ -> event.timestamp }     )`

✅ **`event.timestamp` 값을 기준으로 `Event Time`을 설정**  
✅ **최대 `5초`까지 지연된 데이터(Out-of-Order)를 허용**

---

# **🚀 4️⃣ Watermark 전략별 차이**

|Watermark 전략|설명|
|---|---|
|**`forMonotonousTimestamps()`**|이벤트가 **정확한 시간 순서대로 도착**할 경우|
|**`forBoundedOutOfOrderness(Duration.ofSeconds(5))`**|최대 `5초`까지 늦게 도착한 이벤트를 허용|
|**`forGenerator()`**|커스텀 워터마크 생성 가능|

---

# **🚀 5️⃣ Watermark를 적용한 `Event Time` 기반 윈도우 연산**

## **✅ 예제 2: `Event Time` 기반 윈도우 처리**

kotlin

복사편집

`val result = streamWithTimestamps     .keyBy { it.userId }     .window(TumblingEventTimeWindows.of(Time.seconds(10)))  // ✅ 10초 단위 윈도우     .reduce { event1, event2 ->          event1.count += event2.count         event1     }`

✅ **10초 단위로 `Event Time` 기준으로 데이터 집계**  
✅ **Watermark를 기반으로 윈도우 연산을 수행하여 늦게 도착한 데이터도 처리 가능**

---

# **🚀 6️⃣ Watermark 사용이 필요한 경우**

✅ **1️⃣ 데이터가 지연될 수 있는 환경**

- Kafka, Kinesis, Pulsar 같은 **분산 메시지 시스템에서 스트리밍 데이터가 들어오는 경우**
- 네트워크 지연, 시스템 부하로 인해 이벤트가 늦게 도착할 가능성이 높은 경우

✅ **2️⃣ 정확한 시간 기반 윈도우 연산이 필요한 경우**

- **10초 단위 이벤트 집계** 같은 `Event Time` 기반 분석 수행 시 필요
- `Processing Time`을 사용하면 지연된 데이터가 무시될 수 있음

✅ **3️⃣ 스트리밍 데이터가 순서가 맞지 않는 경우**

- **실제 발생 시간과 Flink에서 수신하는 시간이 다를 경우**
- 예: 모바일 앱에서 사용자의 행동 이벤트가 **네트워크 상태에 따라 뒤늦게 전송**되는 경우

---

# **🚀 7️⃣ 정리**

|개념|설명|
|---|---|
|**타임스탬프(Timestamp)**|`Event Time`을 사용하기 위해 데이터에서 직접 추출해야 함|
|**워터마크(Watermark)**|데이터가 늦게 도착해도 정확한 시간 기반 연산을 수행하도록 보장|
|**언제 사용해야 하나?**|네트워크 지연, Kafka 같은 메시지 큐 사용 시 필요|
|**Flink에서 어떻게 설정?**|`.assignTimestampsAndWatermarks(WatermarkStrategy)` 사용|

📌 **💡 `Event Time`을 사용하려면 타임스탬프를 직접 설정해야 함**  
📌 **💡 Watermark는 데이터 지연을 허용하면서도 정확한 시간 연산을 수행하기 위해 필요**

✅ **이제 Flink에서 `타임스탬프`와 `워터마크`를 활용하여 시간 기반 스트리밍 분석을 정확하게 수행할 수 있습니다!** 🚀🔥

4o

##### 나의 말:

이벤트 타임으로 워터 마크를 만든다면, 지연이 아닌 실제 데이터가 흘러들어오지 않는 것이 정상 적인 상황이라면 어떻게 해? 예를들면 12시00분 이 마지막 이벤트 타임이고 그 다음 데이터가 12시1분에 다음 이벤트가 발생했는데, 워터마크가 forBoundedOutOfOrderness(Duration.ofSeconds(5)) 로 설정되어 있다면 정상적인 이벤트이지만 누락되는거 아냐?

###### ChatGPT의 말:

### **✅ 이벤트 타임에서 데이터 유입이 간헐적인 경우, 워터마크 문제 해결 방법**

> **🔥 Flink에서 `Event Time` 기반 워터마크를 사용할 때, 데이터가 불규칙하게 들어오는 경우 어떻게 처리할 것인가?**  
> **👉 `Watermark`는 특정 시점 이후 데이터가 없을 경우 갱신되지 않아, 늦게 도착한 이벤트가 누락될 가능성이 있음**

---

# **🚀 1️⃣ 문제 상황**

**💡 이벤트가 불규칙하게 발생하는 경우 발생하는 문제**

1. **마지막 이벤트 타임이 `12:00:00` 이고, 다음 이벤트가 `12:01:00`에 발생하는 경우**
2. **워터마크(`forBoundedOutOfOrderness(Duration.ofSeconds(5))`)는 `12:00:00 - 5초 = 11:59:55`로 설정됨**
3. **`12:00:00` 이후 1분 동안 데이터가 없으면 워터마크가 갱신되지 않음**
4. **그동안 윈도우 연산이 닫혀버리면, `12:01:00`에 도착한 데이터는 누락될 수 있음!** ❌

---

# **🚀 2️⃣ 해결 방법**

✅ **1️⃣ `withIdleness()`를 설정하여 데이터가 없을 때도 워터마크를 갱신**  
✅ **2️⃣ `allowLateness()`를 사용하여 늦게 도착한 데이터를 추가 처리**  
✅ **3️⃣ `WatermarkStrategy.forGenerator()`를 사용하여 커스텀 워터마크 생성**

---

## **✅ 해결 방법 1: `.withIdleness()`로 유휴 스트림 처리**

### **📌 `withIdleness()`를 추가하면 데이터가 없어도 일정 시간 후 워터마크가 갱신됨**

kotlin

복사편집

`val streamWithWatermark = inputStream     .assignTimestampsAndWatermarks(         WatermarkStrategy             .forBoundedOutOfOrderness<MyEvent>(Duration.ofSeconds(5))              .withIdleness(Duration.ofMinutes(1)) // ✅ 1분 동안 데이터가 없으면 워터마크 강제 갱신             .withTimestampAssigner { event, _ -> event.timestamp }     )`

✅ **`withIdleness(Duration.ofMinutes(1))`을 추가하면, 1분 동안 데이터가 없더라도 워터마크가 갱신됨**  
✅ **즉, 데이터가 간헐적으로 들어와도 워터마크가 계속 업데이트되므로 누락 방지 가능**

---

## **✅ 해결 방법 2: `allowLateness()`로 늦게 도착한 데이터 처리**

### **📌 윈도우가 닫힌 후에도 데이터를 처리할 수 있도록 허용**

kotlin

복사편집

`val result = streamWithWatermark     .keyBy { it.userId }     .window(TumblingEventTimeWindows.of(Time.minutes(1))) // 1분 단위 윈도우     .allowedLateness(Time.seconds(30)) // ✅ 윈도우 종료 후 30초까지 늦은 데이터 허용     .reduce { event1, event2 ->         event1.count += event2.count         event1     }`

✅ **이제 `12:01:00`에 도착한 데이터가 `12:00:00` 윈도우에 포함될 수 있음**  
✅ **윈도우가 닫힌 후 `30초` 동안 추가 데이터를 받아들일 수 있음**

---

## **✅ 해결 방법 3: `forGenerator()`를 사용하여 커스텀 워터마크 생성**

### **📌 직접 워터마크를 생성하여 유휴 상태에서도 갱신**

kotlin

복사편집

`val watermarkStrategy = WatermarkStrategy     .forGenerator<MyEvent> { ctx ->         object : WatermarkGenerator<MyEvent> {             var maxTimestamp = Long.MIN_VALUE              override fun onEvent(event: MyEvent, eventTimestamp: Long, output: WatermarkOutput) {                 maxTimestamp = maxOf(maxTimestamp, eventTimestamp)                 output.emitWatermark(Watermark(maxTimestamp - 5000)) // ✅ 5초 지연 허용             }              override fun onPeriodicEmit(output: WatermarkOutput) {                 output.emitWatermark(Watermark(maxTimestamp - 5000)) // ✅ 주기적으로 워터마크 갱신             }         }     }`

✅ **커스텀 워터마크 생성기로 직접 타임스탬프를 조절 가능**  
✅ **이벤트가 간헐적으로 발생해도 주기적으로 워터마크를 강제로 갱신 가능**

---

# **🚀 3️⃣ 최종 정리**

|해결 방법|적용 방법|설명|
|---|---|---|
|**유휴 상태에서도 워터마크 갱신**|`.withIdleness(Duration.ofMinutes(1))`|✅ 데이터가 없을 때도 워터마크를 업데이트|
|**늦게 도착한 데이터 처리**|`.allowedLateness(Time.seconds(30))`|✅ 윈도우가 닫힌 후에도 일정 시간 동안 추가 데이터 허용|
|**커스텀 워터마크 생성**|`.forGenerator()` 사용|✅ 이벤트가 없어도 워터마크를 주기적으로 갱신|

📌 **💡 데이터가 간헐적으로 들어올 때는 `withIdleness()`를 추가하여 워터마크를 강제 갱신**  
📌 **💡 `allowLateness()`를 설정하면 윈도우가 닫혀도 늦게 도착한 데이터를 수용 가능**  
📌 **💡 `forGenerator()`를 사용하면 더욱 세밀하게 워터마크를 제어 가능**

✅ **이제 이벤트 타임에서 데이터 유입이 불규칙한 경우에도 안전하게 처리할 수 있습니다!** 🚀🔥