### `print()` 와 `sink()` 차이점

1. **`print()`**:
    
    - Flink의 기본 출력 연산자로, 변환된 데이터를 콘솔에 출력합니다.
    - 디버깅 목적으로 사용되며, 파일이나 데이터베이스 등과 같은 외부 시스템으로 데이터를 저장하지 않습니다.
    - 예제:
```kotlin
transformedStream.print()
```
    
2. **`sink()`**:
    
    - 데이터를 외부 시스템(예: 파일, 데이터베이스, 메시지 큐 등)으로 내보낼 때 사용됩니다.
        
    - 다양한 sink 연산자가 있으며, `addSink()` 메서드를 통해 커스텀 싱크를 추가할 수 있습니다.
        
    - 예제(파일로 데이터 방출):
        ```kotlin
transformedStream.writeAsText("/path/to/output.txt")
```
    - Kafka, MySQL, ElasticSearch 등으로 데이터를 보낼 수도 있습니다.    
```kotlin
transformedStream.addSink(MyCustomSinkFunction())
```
### 차이점 요약:

|기능|`print()`|`sink()`|
|---|---|---|
|용도|디버깅, 테스트|외부 저장소로 방출|
|출력 위치|콘솔|파일, DB, 메시지 큐 등|
|사용 방법|`print()`|`writeAsText()`, `addSink()` 등|

따라서, 단순한 테스트에는 `print()`를 사용하고, 실제 스트리밍 애플리케이션에서 데이터를 저장하거나 전송하려면 `sink()`를 사용하는 것이 적절합니다.