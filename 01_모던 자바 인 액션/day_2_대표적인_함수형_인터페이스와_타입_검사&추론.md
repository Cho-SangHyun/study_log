## 대표적인 함수형 인터페이스
- `java.util.function` 패키지에 아래와 같이 대표적으로 많이 쓰이는 형태의 함수형 인터페이스들이 있다
<br>  

### Predicate
- 제네릭 형식 T의 객체를 인수로 받아 불리언을 리턴
```java
@FunctionalInterface
public interface Predicate<T> {
    boolean test(T t);
}
```

### Consumer
- 제네릭 형식 T의 객체를 인수로 받아 `void`를 리턴
- 즉 인자를 받아 사용(consume)하는 형태
```java
@FunctionalInterface
public interface Consumer<T> {
    void accept(T t);
}
```

### Function
- 제네릭 형식 T의 객체를 인수로 받아 제네릭 형식 R의 객체를 리턴
- input, output이 있는 전형적인 수학적 의미의 함수(function)를 뜻함
```java
@FunctionalInterface
public interface Function<T, R> {
    R Apply(T t);
}
```

### Supplier
- `void`를 인수로 받아 제네릭 형식 T의 객체를 리턴
- 즉 특정 타입의 객체를 공급(supply)하는 형태
```java
@FunctionalInterface
public interface Supplier<T> {
    T get();
}
```

<br>  

## 람다의 타입 검사, 타입 추론
### 타입 검사
- 람다가 사용되는 `컨텍스트`를 이용해 람다의 타입을 추론할 수 있음
  - 컨텍스트 = 람다가 전달될 메서드 파라미터 또는 람다가 할당되는 변수
- 컨텍스트에서 기대되는 람다 표현식의 형식 = `대상 형식`이라 부름
  - 걍 메서드 시그니처에 쓰여있는 함수형 인터페이스의 타입을 말함

<br>  

예를 들어 다음 코드가 있다 하자
```java
List<Apple> heavyApples = filter(appleInventory, (Apple apple) -> apple.getWeight() > 1000);
```

그러면 타입 검사는 다음처럼 진행된다
1. 우선 filter 메서드의 시그니처를 확인
```java
filter(List<Apple> appleInventory, Predicate<Apple> p)
```
2. 메서드 시그니처 보니까 두 번째 파라미터가 `Predicate<Apple>` 타입이네? 이게 바로 `대상 형식`.
3. Predicate는 test라는 추상 메서드가 있는데, 얘는 Apple을 인자로 받아서 boolean을 리턴하는 함수 디스크립터를 묘사하네?
4. 그러면 heavyApples를 선언할 때 쓰이는 람다도 `(Apple apple) -> boolean` 형식이어야겠는걸?
5. 실제로 람다 `(Apple apple) -> apple.getWeight() > 1000)`은 `(Apple apple) -> boolean` 형식이군!
6. 타입 검사 완료

<br>  

대상 형식에 의거해서 타입을 검사하므로, 똑같은 람다 표현식이 서로 다른 함수형 인터페이스로 쓰일 수 있다.  
예를 들어 `() -> 100`이라는 람다 표현식은 `Supplier`와도 매칭될 수 있고 `Callable`과도 매칭될 수 있음

<br>  

### 타입 추론
- 자바 컴파일러는 람다 표현식이 사용된 컨텍스트(=대상 형식)을 이용해 람다 표현식과 관련된 함수형 인터페이스를 추론
- 즉 대상 형식을 이용해 해당하는 함수형 인터페이스가 갖는 추상 메서드의 함수 디스크립터를 알 수 있고
- 이를 통해 그 부분에 어떤 시그니처를 갖는 람다가 쓰일 건지 `추론`할 수 있다
- 결과적으로 `컴파일러는 람다 표현식의 파라미터 타입에 접근 가능하니 람다 문법에서 파라미터 부분의 타입을 생략해도 된다는 것!`

<br>  

뭔 말이냐면.. 다음 코드가 있다고 하자
```java
List<Apple> heavyApples = filter(appleInventory, (Apple apple) -> apple.getWeight() > 1000);
```

이거를 다음처럼 해도 된다는 거다.
```java
List<Apple> heavyApples = filter(appleInventory, apple -> apple.getWeight() > 1000);
```

람다 표현식의 파라미터 부분의 apple의 타입이 명시적으로 지정되지 않았지만, 타입 추론에 의해 괜찮다는 것.  
물론 이게 항상 좋은 건 아니다. 개발자인 우리 스스로가 어떨 때가 더 가독성이 좋을지를 판단해야 함
