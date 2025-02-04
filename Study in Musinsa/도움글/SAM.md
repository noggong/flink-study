# (1) 람다 표현식으로 구현

```java
@FunctionalInterface 
interface Operator { 
	int calculate(int a, int b); // 단 하나의 추상 메소드 
} 

public class Main { 
	public static void main(String[] args) { 
	// 람다 표현식을 사용한 구현 
	Operator add = (a, b) -> a + b; 
	Operator multiply = (a, b) -> a * b; 
	
	// 계산 수행 
	System.out.println(add.calculate(5, 3)); // 8
	System.out.println(multiply.calculate(5, 3)); // 15 
	} 
}
```
- `Operator`는 단일 추상 메소드 `calculate`를 포함하고 있으므로 **SAM 인터페이스**입니다.
- 람다 표현식을 통해 `add`와 `multiply`의 동작을 간단히 정의할 수 있습니다.
# (2) 메소드 참조 사용
```java
import java.util.function.Consumer;

public class Main {
    public static void main(String[] args) {
        // 메소드 참조 사용
        Consumer<String> printer = System.out::println;

        // 실행
        printer.accept("Hello, SAM!"); // "Hello, SAM!" 출력
    }
}
```

- `Consumer`는 Java에서 제공하는 함수형 인터페이스(SAM)로, `accept`라는 단일 메소드를 포함합니다.
- `System.out::println`은 메소드 참조로 `Consumer`의 `accept` 메소드를 구현합니다.

# 장점
- 람다식과 메소드 참조를 사용하면 익명 클래스 작성에 비해 코드가 간결해지고, 가독성이 향상됩니다.
- SAM 인터페이스는 함수형 프로그래밍 스타일을 지원하여 복잡한 연산을 간단히 표현할 수 있게 합니다.
# 결론
- 이 코멘트는 **SAM 인터페이스가 단일 추상 메소드만 포함하기 때문에, 람다 표현식이나 메소드 참조로 구현이 가능하다**는 점을 설명한 것입니다. 이를 통해 Java 개발자는 코드의 간결함과 유연성을 얻을 수 있습니다.