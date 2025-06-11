## 그룹화
- 데이터 집합을 하나 이상의 특성으로 분류해서 그룹화하는 연산
- 명령형으로 구현하면 개빡세지만 스트림에서는 함수형으로 간단히 구현 가능
- `collect`의 인자로 `Collectors.groupingBy`를 줘서 구현할 수 있다

ex)
```java
Map<Dish.Type, List<Dish>> dishesByType = menu.stream().collect(Collectors.groupingBy(Dish::getType));

// result : {FISH=[prawns, salmon], MEAT=[pork, beef, chicken], ...}
```

- 여기서 `Collectors.groupingBy`의 첫 번째 인자로 쓰인 `Dish::getType`를 분류 함수(classifier)라 부른다

<br>  

### 그룹화된 요소 조작
- 요소를 그룹화한 뒤 각 그룹의 요소를 조작하는 연산이 필요할 때도 있다
  - 예를 들면 Dish의 타입으로 그룹화한 후 각 그룹 안에서 필터링
- 500칼로리가 넘는 애들을 필터링한 뒤 타입으로 그룹화한다고 하면 다음 코드를 쓸 수 있었다

```java
Map<Dish.Type, List<Dish>> heavyDishesByType = menu.stream()
        .filter(dish -> dish.getCalories() > 500)
        .collect(Collectors.groupingBy(Dish::getType));
```

- 근데 이렇게 하면 필터링 단계에서 타입이 MEAT인 애들이 다 걸려졌다고 가정하면 결과 Map에는 키를 MEAT로 갖는 그룹이 없다.
- `Collectors.groupingBy`는 이런 상황을 위해 두 번째 인자를 받는 "오버로드된 버전"도 제공
  - 두 번째 인자로 `Collectors.filtering`라는 `Collector`를 준다. 이 놈은 필터링에 쓰일 `Predicate`와 결과물의 형태인 `Collector`의 구현체를 인자로 받는다

```java
Map<Dish.Type, List<Dish>> heavyDishesByType = menu.stream()
        .collect(Collectors.groupingBy(Dish::getType,
                Collectors.filtering(dish -> dish.getCalories() > 500, Collectors.toList())));

// Collectors.filtering의 첫째 인자 : dish -> dish.getCalories() > 500 -> 필터링에 사용될 첫 번째 Predicate
// Collectors.filtering의 둘째 인자 : Collectors.toList -> 필터링된 애들의 결과물 (리스트로 뽑아내겠다)
// 즉 타입으로 그루핑한후, 각 그룹(서브스트림)안에서 필터링한 결과물들을 리스트로 뽑아낸다는 의미가 된다
```

- 비슷하게 `Collectors.mapping`, `Collectors.flatMapping`도 존재하며 각각 그루핑된 그룹 안에서 매핑을 한다는 의미

<br>  

### 다수준 그룹화
- 그룹 안에서 그루핑하고 또 그 안에서 그루핑하고 그런 거 말함
- 그룹화된 요소 조작 편에서 봤듯이, 두 개의 인자를 받는 `Collectors.groupingBy`는 첫 번째 인자로 `classfier`, 두 번째 인자로 `Collector`를 받는다.
  - 사실 `classifier` 하나만 받는 `Collectors.groupingBy`도 내부적으로 다음과 같이 두 개의 인자를 받는 `Collectors.groupingBy`를 호출 

```java
public static <T, K> Collector<T, ?, Map<K, List<T>>>
groupingBy(Function<? super T, ? extends K> classifier) {
    // Collectors.toList를 넘기는 형태. 즉 결과물을 리스트로 뽑아내란 의미죠
    return groupingBy(classifier, toList());
}
```

- `Collectors.groupingBy`도 `Collector`이므로 두 번째 인자로 `Collectors.groupingBy`를 넘길 수 있고, 그리 되면 다수준 그룹화가 가능해진다

```java
Map<Dish.Type, Map<String, List<Dish>>> dishesByTypeCaloricLevel = menu.stream()
        .collect(Collectors.groupingBy(Dish::getType,
                Collectors.groupingBy(dish -> {
                        if (dish.getCalories() > 500) return "HEAVY";
                        return "NORMAL";
                    }
                ))
        );
```

<br>  

### 서브그룹으로 데이터 수집
- 앞서 설명했듯 두 개의 인자를 받는 `Collectors.groupingBy`는 첫 번째 인자로 `classfier`, 두 번째 인자로 `Collector`를 받는다
- 두 번째 인자로 넘기는 `Collector` 형식은 `classfier`로 분류된 그룹(서브스트림)에서 어떤 형태의 결과물을 뽑아내겠다고 지정하는 역할.
- 즉 제한이 없음. 다음처럼 해도 가능하다는 얘기

```java
Map<Dish.Type, Long> countPerTypes = menu.stream().collect(Collectors.groupingBy(Dish::getType, Collectors.counting()));
```
  
- 당연히 다음처럼 해도 가능

```java
Map<Dish.Type, Optional<Dish>> maxCaloricDishByTypes = menu.stream()
        .collect(Collectors.groupingBy(Dish::getType, Collectors.maxBy(comparingInt(Dish::getCalories))));
```

<br>  

### 컬렉터 결과를 다른 형식에 적용하기
- 다음 코드는 `Collectors.groupingBy`의 두 번째 인자로 준 `Collector`로 인한 결과물이 `Optional`이다.

```java
Map<Dish.Type, Optional<Dish>> maxCaloricDishByTypes = menu.stream()
        .collect(Collectors.groupingBy(Dish::getType, Collectors.maxBy(comparingInt(Dish::getCalories))));
```

- 근데 `Collector`로 인한 결과물을 다른 형식으로 활용하고 싶다면? `Collectors.collectingAndThen`을 쓸 수 있다

```java
Map<Dish.Type, Dish> maxCaloricDishByTypes = menu.stream()
        .collect(Collectors.groupingBy(Dish::getType
                Collectors.collectingAndThen(
                        Collectors.maxBy(comparingInt(Dish::getCalories)),
                        Optional::get
                )
        ));
```

- `collectingAndThen`. 말 그대로 컬렉팅한 결과를 Then한다.
  - `Collectors.maxBy`로 컬렉팅한 결과 `Optional<Dish>`에 대해 `Optional::get`을 먹인 것을 사용하겠다!
  - 즉 `Optional::get`이 일종의 변환 함수
