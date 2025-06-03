## 컬렉터란?
- 이번 장에서 다룰 것은 `collect`라는 최종연산도 다양한 요소 누적 방식을 인자로 받아서 스트림을 최종 결과로 도출하는 "리듀싱 연산"을 수행 가능하다는 것
- `Collector` 인터페이스의 구현이 스트림의 요소를 어떤 식으로 도출할지 지정하는 역할을 한다
  - `Collectors.toList`는 `Collectors`의 static method지만 `Collector` 인터페이스의 구현으로 "각 요소를 리스트로 만들어라"라는 역할을 한다
- 컬렉터의 최대 장점 : `collect`로 결과를 수집하는 과정을 간단히 정의할 수 있다는 것 (함수형 API의 장점이기도)
  - 스트림에 `collect`를 호출하면 스트림의 요소에 컬렉터로 파라미터화된 리듀싱 연산(`collect`가 인자로 받은거)이 수행됨

<br>  

## 리듀싱과 요약
- `collect`의 인자로 `Collectors.toList`를 넘겼을 떄의 예에서 알 수 있듯이, 컬렉터로 스트림의 항목을 컬렉션으로 재구성 가능
- 얘도 일종의 리듀싱 연산임. 스트림의 모든 항목을 하나의 결과로 만든 거니까
- 즉 리듀싱 연산은 `sum`처럼 하나의 "값"으로 만드는 것 뿐만 아니라 하나의 결과로 만드는 것을 얘기하는 것이므로, 여러 뎁스가 있는 `Map` 등으로도 리듀싱 연산을 해줄 수 있음

<br>

### 스트림에서 최대 최소 검색
- `reduce` 메서드를 통해서 최댓값 찾을 땐 다음처럼 했었다. 참고로 `reduce`로 최종연산
```java
int maxCalories = menus.stream().map(Dish::getCalories).reduce(0, (a, b) -> Math.max(a, b));
// 다르게 표현하면
Optional<Integer> maxCalories = menus.stream().map(Dish::getCalories).reduce(Integer::max);
```

- 근데 이걸 `collect`에서도 컬렉터를 통한 리듀싱 연산으로 처리 가능함
- `Collectors`의 또 다른 static method `maxBy`가 있고, 이 놈은 인자로 `Comparator`를 받는다. 다음처럼 활용 가능
```java
Comparator<Dish> dishCaloriesComparator = Comparator.comparingInt(Dish::getCalories);

Optional<Dish> maxCalorieDish = menu.stream().collect(Collectors.maxBy(dishCaloriesComparator));
```

- 참고로 menu가 비워져있을 수도 있으니 `Optional`을 리턴하게 되는 것

<br>

### 요약 연산
- 스트림의 요소들이 가지고 있는 숫자 필드들의 합계, 평균 등을 산출해야 할 때도 있는데, 이떄도 사용 가능한 리듀싱 연산이 제공되며 이를 요약(Summarization) 연산이라 부름
- 대표적으로 요약 연산 중 하나로 `Collectors`는 또다른 static method `summingInt`를 제공
  - 객체를 `int`로 매핑하는 `ToIntFunction`이란 함수형 인터페이스를 인자로 받아서, 이 놈을 활용해 객체를 `int`로 매핑한 컬렉터를 리턴
- 다음처럼 활용 가능하다.
```java
int totalCalories = menu.stream().collect(Collectors.summitInt(Dish::getCalories));
```

- 이 코드는 다음과 동치다.
```java
int totalCalories = menu.stream().mapToInt(Dish::getCalories).sum();
```

<br>

### 리듀싱 연산 - joining을 활용한 문자열 연결
- `Collectors`는 또 다른 static method `joining`을 제공하며, 이 놈은 모든 문자열을 하나의 문자열로 합치는 리듀싱 연산을 해준다
```java
String allNames = names.stream().collect(Collectors.joining(", "));
```

<br>

### 범용 리듀싱 요약 연산
- 사실 지금까지 살펴본 모든 컬렉터는 `reducing` 팩토리 메서드로 정의할 수 있음
- 예를 들어 모든 칼로리 총합을 구하는 예제는 `Collectors.reducing`으로도 다음처럼 구할 수 있다.
```java
int totalCalories = menu.stream().collect(Collectors.reducing(
        0, Dish::getCalories, (i, j) -> i + j));
```
- 파라미터 3개를 받는 `Collectors.reducing`을 뜯어보면
  - 첫 번째 인자 : 리듀싱 연산의 시작값. 스트림에 요소가 없을 땐 반환값
  - 두 번째 인자 : 변환 함수 (매핑 함수)
  - 세 번짜 인자 : 같은 타입의 두 놈을 짝짝꿍해서 하나로 리턴하는 `BinaryOperator`

- 또 다른 예로 문자열들을 합치는 joining 연산도 `Collectors.reducing`으로 다음처럼 구할 수 있다.
```java
String allNames = names.stream().collect(Collectors.reducing(
    "",
    (name1, name2) -> name1 + (name1.isEmpty() ? "" : ", ") + name2
));
```

<br>

### collect vs reduce
- 근데 솔까 `collect`에 `Collectors.reducing`을 통해 joining연산했던 거 `reduce`로도 다음처럼 할 수 있다.
```java
String allNames = names.stream().reduce(
    "",
    (s1, s2) -> s1 + ", " + s2
);
```

- 이 둘의 차이는 "의미론적인 문제"와 "실용성 문제"라는 두 가지다.
- 의미론적 관점에서는 
  - `collect`는 도출하려는 결과를 담는 컨테이너(= 자료구조)를 바꾸도록 설계된 메서드
  - `reduce`는 두 값을 하나로 도출하는 불변형 최종연산
- 실용성 문제 관점에선 `reduce`가 병렬성 관점에서 더 안 좋음, 다만 가독성은 더 좋을 수 있음
