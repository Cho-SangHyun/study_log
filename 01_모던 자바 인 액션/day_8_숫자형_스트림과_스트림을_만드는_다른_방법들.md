## 숫자형 스트림
- 보통 숫자의 모음(리스트)를 다룰 땐 그들의 합(sum), 최댓값(max), 최솟값(min) 등을 구하는 일이 多
- 근데 `Stream`은 `sum`이나 `max`같은 메서드는 제공 안함. 왜냐면 스트림 요소가 `Human`이런 거면 `sum`을 어케 정의해야 할지 모르니까
- 그래서 `Stream`은 `IntStream`, `DoubleStream`, `LongStream`이라는 기본형 스트림을 별도로 제공
- 이 놈들은 `sum`, `max`, `min` 등이 정의되어 있다!
- 기존에 `Stream<Integer>`를 그대로 쓰는 건 박싱 비용(int -> Integer)과 언박싱 비용(Integer -> int)이 발생하는데, `IntStream`은 기본 타입 `int`를 그대로 사용하는 놈이라 그런 비용을 줄이는 효율성도 가짐
  - 참고 : 잦은 박싱과 언박싱은 GC 부담을 증가시키기도 한다   

예)  

```java
int totalCalories = menus.stream()
        .mapToInt(Dish::getCalories) // IntStream을 반환
        .sum();
```

<br>  

- 기본형 스트림은 `boxed`라는 메서드도 제공하는데, 이 놈은 기본형 스트림을 일반 스트림으로 바꿔주는 역할을 함

예)

```java
IntStream intStream = menus.stream().mapToInt(Dish::getCalories);
Stream<Integer> normalStream = intStream.boxed();
```

<br>  

### 기본형 스트림으로 숫자 범위 다루기
- 기본형 스트림 중 `IntStream`과 `LongStream`은 `range`, `rangeClosed`라는 정적 메서드도 제공
- 둘 다 인자로 주어진 두 값의 범위에 해당하는 값들의 스트림을 만드는 거
- `range`는 개구간(끝값 포함 x), `rangeClosed`는 폐구간(끝값 포함 o) 스트림을 만든다는 차이다


예)

```java
IntStream evenNumbers = IntStream.rangeClosed(1, 100) // 1부터 100까지의 수들
        .filter(n -> n % 2 == 0);                     // 짝수만 거르기

System.out.println(evenNumbers.count());              // 50

```

<br>  

## 스트림을 만드는 다른 방법들
### 1. 값으로 만들기
- `Stream.of`를 사용해 값으로 스트림을 만들 수 있다.
  
```java
Stream<String> streamByValue = Stream.of("Hi", "Nice", "To", "Meet", "You");
```

<br>  

### 2. nullable한 객체로 만들기
- `Nullable`한(`Null`이 될 수 있는) 객체에 대한 스트림을 만드는 경우가 필요할 때 쓴다
- `Stream.ofNullable`을 사용한다

```java
Stream<String> streamByNullable = Stream.ofNullable(human);
```

- 이는 다음과 동치이다

```java
Stream<String> streamByNullable = human == null ? Stream.empty() : Stream.of(human);
```

<br>  

### 3. 배열로 만들기
- 배열을 인자로 받는 Arrays.stream을 사용해 배열을 원천으로 하는 스트림도 만들 수 있다. 예제는 생략

<br>  

### 4. 파일로 만들기
- 파얼 처리 등의 I/O 연산에 사용하는 NIO API도 스트림 API를 사용 가능하다.
- `Stream`은 `AutoCloseable` 인터페이스를 구현하므로 `try-with-resources` 문법도 사용 가능함

<br>  

### 5. 함수로 무한 스트림 만들기
- `Stream`은 함수에서 스트림을 만들 수 있는 `Stream.iterate`와 `Stream.generate`라는 두 정적 메서드를 제공
- 두 연산은 무한스트림(언바운드 스트림)이라 불리는, 크기가 제한되지 않은 스트림을 만든다.
  - 단 컬렉션처럼 모든 값을 계산해서 메모리에 들고 있는게 아니라 필요한 순간에 필요한 값들만 계산해서 들고 있는 방식
- 두 정적 메서드로 만든 스트림은 요청할 때마다 주어진 함수를 이용해 값을 만듦
- 보통은 무한한 값을 출력하지 않도록 `limit`, `takeWhile` 등을 버무려 사용

<br>  

#### iterate
```java
Stream.iterate(0, n -> n + 2)
        .limit(5)
        .forEach(System.out::println); // 0, 2, 4, 6, 8
```

- 위 예제처럼 초깃값(예제에선 0)과 람다를 인자로 받음
- 인자로 받는 람다는 `UnaryOperator<T>` 타입 (T타입을 인자로 받아 T타입을 리턴하는 함수형 인터페이스)
- 계산된 기존 결과가 계속해서 람다식의 인자로 들어오는 형태

<br>  

#### generate
```java
Stream.generate(Math::random)
        .limit(5)
        .forEach(System.out::println); 
```

- 람다를 인자로 받으며, 이 놈은 `Supplier<T>` 타입 (void를 인자로 T타입을 리턴하는 함수형 인터페이스)
- 그때그때 람다식을 실행한 결과물을 스트림의 요소로 만드는 형태
