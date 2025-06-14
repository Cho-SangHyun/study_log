## 스트림 분할
- 분할함수(partitioning function)이라 불리는 `Predicate`를 사용하는 특수한 그룹화 기능으로 `Collectors.partitioningBy`를 말함
- `Collectors.groupingBy`랑 비슷하나, `Predicate`는 불리언을 반환하므로 맵의 키 형식은 `Boolean` (즉 true or false)

예) 음식을 채식 요리와 채식이 아닌 요리로 그룹화
```java
Map<Boolean, List<Dish>> partitionedMenus = menus.stream().collect(Collectors.partitioningBy(Dish::isVegetarian));

// 결과 : {false=[pork, beef], true=[rice, french fries]}
```

- `Collectors.groupingBy`처럼 `Collectors.partitioningBy`도 파라미터를 두 개 받는 오버로드된 메서드가 있음
  - 이 놈은 두 번째 파라미터로 `Collector`를 받음. 이를 통해 파티셔닝된 그룹 안에서 또 어떤 리듀싱 연산을 해줄지 정의해줄 수 있음
  - 예를 들면 파티셔닝된 거 안에서 그룹핑 또는 파티셔닝 등..
  - 파라미터 1개만 받는 놈도 내부적으론 `Collectors.toList()`를 인자로 넘기는 `partitioningBy`를 호출

```java
public static <T>
Collector<T, ?, Map<Boolean, List<T>>> partitioningBy(Predicate<? super T> predicate) {
    return partitioningBy(predicate, toList());
}
```

<br>  

## Collector 인터페이스
- `Collector` 인터페이스의 구현 = 스트림의 요소를 어떤 식으로 도출할지 지정하는 역할
- 즉 "`Collector` 인터페이스 = 리듀싱 연산 방법"으로 이해할 수 있다
- `Collector` 인터페이스는 그 리듀싱 연산을 어떻게 구현할지 제공하는 가상 메서드 집합으로 구성. `Collector`의 시그니처와 가상 메서드 집합은 다음과 같음

```java
public interface Collector<T, A, R> {
    Supplier<A> supplier();
    BiConsumer<A, T> accumulator();
    BinaryOperator<A> combiner();
    Function<A, R> finisher();
    Set<Characteristics> characteristics();
}
```

> T : 수집될 스트림 요소들의 제네릭 타입  
> A : 누적자, 즉 수집 과정에서 누적되는 중간 결과 객체의 형식  
> R : 수집 연산 결과 객체 형식(대개 컬렉션이 많이 쓰인다)
>
> supplier ~ finisher는 Stream의 collect 메서드에서 실행되는 함수들(리듀싱 연산을 하는)를 반환하는 가상 메서드
> characteristics는 Stream의 collect 메서드가 어떤 최적화(병렬화 등)을 사용해 리듀싱 연산을 수행할 것인지 돕는 역할

- 예를 들어 Stream<T>의 모든 요소들을 List<T>로 수집하는 커스텀 Collector를 구현한다고 하면 다음처럼 해줄 수 있다

```java
public class CustomToListCollector<T> implements Collector<T, List<T>, List<T>>
```

<br>  

### supllier : 새로운 결과 컨테이너(누적자 객체) 만들기
- `Collector`의 `suppler`는 빈 결과로 이루어진 `Supplier`를 리턴해야 함
  - 참고) `Supplier`는 void를 인수로 받아 제네릭 형식 T의 객체를 리턴하는 함수형 인터페이스 (일종의 보급)
- 즉 `Collector`의 `suppler`는 수집 과정에서 빈 누적자 인스턴스를 만드는 파라미터 없는 메서드

예)
```java
public Supplier<List<T>> supplier() {
    return () -> new ArrayList<T>();
}
```

<br>  

### accumulator : 결과 컨테이너(누적자 객체)에 요소 추가
- `Collector`의 `accumulator`는 리듀싱 연산을 수행하는 `BiConsumer`를 리턴해야 함
  - 참고) `Consumer`는 제네릭 형식 T의 객체를 인수로 받아 void를 리턴하는 함수형 인터페이스 (즉 인자를 받아 어떤 연산을 수행하는 형태)
  - `BiCinsumer`는 인자를 두 개 받아 이를 사용한 어떤 연산을 수행하는 함수형 인터페이스
- 해당 BiConsumer는 다음과 같은 두 인자를 적용
  1. 첫 번째 인자 : 스트림의 n - 1번째 요소까지 수집한 누적자
  2. 두 번째 인자 : n번째 요소

예)
```java
public BiConsumer<List<T>, T> accumulator() {
    return (list, item) -> list.add(item);
}
```

<br>  

### finisher : 최종 변환값을 결과 컨테이너로 적용
- `Collector`의 `finisher`는 스트림 탐색을 끝내고 누적자 객체를 최종 결과로 변환하며 누적 과정을 종결시키는 `Function`을 리턴해야 한다
  - 참고) `Function`은 제네릭 형식 T의 객체를 인수로 받아 제네릭 형식 R의 객체를 리턴하는 함수형 인터페이스 (인풋과 아웃풋이 있는 형태)
  
예)
```java
public Function<List<T>, List<T>> finisher() {
    return list -> list;
    // 위처럼 변환과정이 필요없다면 Function.identity();를 해도 됨
}
```

<br>  

### combiner : 두 결과 컨테이너 병합
- `Collector`의 `combiner`는 스트림의 서로 다른 서브파트를 병렬 처리할 때 누적자가 이 결과를 어떻게 처리할지 정의하는 `BinaryOperator`를 리턴해야 한다
  - 참고) `BinaryOperator`는 `BiFunction`(두 인풋을 받는 `Function`)의 일종으로, 두 인풋과 아웃풋의 타입이 모두 같은 함수형 인터페이스 

예)
```java
public BinaryOperator<List<T>> combiner() {
    return (list1, list2) -> {
        list1.addAll(list2);
        return list1;
    }
}
```

- `Collector`의 `combiner`를 활용 시 리듀싱을 병렬로 수행 가능

<br>  

### characteristics : 최적화 힌트 제공
- `Collector`의 `characteristics`는 컬렉터의 연산을 정의하는 `Characteristics` 타입의 불변 집합을 리턴해야 한다
  - 집합이라는 것은 `Characteristics` 타입의 값이 여러 개 있을 수 있다는 거  
  - 참고) `Characteristics`는 스트림을 병렬로 리듀스할건지, 병렬 리듀스를 한다면 어떤 최적화를 선택할지 힌트를 제공하는 `enum`
  - 다음 3가지를 포함한다

```java
enum Characteristics {
    UNORDERED,
    CONCURRENT,
    IDENTITY_FINISH
}
```

- UNORDERD : 리듀싱 결과는 스트림 요소를 방문한 순서와 누적 순서에 영향 안 받는다. 즉 입력 스트림의 요소 순서를 고려하지 않아도 됨을 의미
- CONCURRENT : 멀티 스레드에서 `Collector`의 `accumulator`를 동시 호출 가능하며 이 컬렉터는 스트림의 병렬 리듀싱을 수행 가능하다. 즉 병렬 스트림에서 안전하게 병합할 수 있다는 의미
  - UNORDERD가 설정되지 않았다면 데이터 소스가 정렬되어 있지 않은 상황(집합처럼 요소 순서가 무의미한)에서만 병렬 리듀싱 수행 가능
  - 즉 UNORDERED도 함께 설정되어야 병렬로 자유롭게 병합 가능
- IDENTITY_FINISH : 중간 누적결과와 최종결과가 같음을 의미
  - 즉 `Collector`의 `finisher` 메서드가 자기 자신을 반환함을 나타냄

