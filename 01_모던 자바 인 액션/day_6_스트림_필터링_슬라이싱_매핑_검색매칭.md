## 1. 필터링
### 프레디케이트로 필터링
- 스트림의 `filter` 메서드는 인자로 `Predicate(객체를 인자로 받아 불리언을 반환)`를 받아 조건에 일치하는 모든 요소를 포함하는 스트림을 반환
```java
List<Dish> vegetarianMenus = menus.stream()
        .filter(Dish::isVegetarian)
        .collect(toList());
```

### 고유 요소 필터링 (중복 필터링)
- 스트림의 중복된 요소들을 필터링해 고유 요소만 반환하는 `distinct` 메서드도 제공함
- 고유 여부 판단 기준 = 스트림에서 만든 객체의 `hashCode`, `equals`로 결정
```java
List<Integer> nums = Arrays.asList(1, 2, 1, 3, 3, 2, 5);
nums.stream()
        .distinct()
        .forEach(System.out::print); 
```

<br>  

## 2. 슬라이싱
- 스트림의 요소를 선택 또는 스킵하는 방법들, 구체적으론 처음 몇 개를 무시하거나 특정 크기로 스트림을 줄이는 방법들이다.
- 자바 9부터 지원

### 프레디케이트를 이용한 슬라이싱
- 예를 들어 정수 리스트가 오름차순 정렬되어 있을 때 100보다 낮은 애들을 뽑아내야 된다고 하자.
- 외부 반복을 사용하면 100이 넘는 시점에서 `break`를 걸면 불필요한 요소들에 대해 for로 순회하는 걸 막을 수 있다.
- 하지만 스트림의 `filter`만으로는 100이 넘는 지점에서 중단할 수 없다
- 이런 상황을 위해 스트림은 `takeWhile`, `dropWhile`을 제공한다

#### 1) takeWhile
- `filter`처럼 인자로 `Predicate`를 받으며, 해당 `Predicate`가 False가 되는 요소에서 내부 반복을 멈춘다.
- 즉 `Predicate`가 거짓이 될 때까지 내부 반복을 진행하며 그 때가지 참인 애들만 뽑아내는 역할
- 거짓이 될 때까지 지나가는 애들은 모두 챙기겠다
```java
List<Integer> nums = Arrays.asList(44, 55, 66, 100, 102, 205, 300);
List<Integer> smallNums = nums.stream()
        .takeWhile(num -> num < 100)
        .collect(toList()); // 44, 55, 66
```

#### 2) dropWhile
- `filter`처럼 인자로 `Predicate`를 받으며, `takeWhile`의 반대역할이다
- 즉 `Predicate`가 거짓이 될 때까지 내부 반복을 진행하며 그 때참인 애들을 "걸러내는" 역할
- 거짓이 될 때까지 지나가는 애들은 모두 버리겠다
```java
List<Integer> nums = Arrays.asList(44, 55, 66, 100, 102, 205, 300);
List<Integer> smallNums = nums.stream()
        .dropWhile(num -> num < 100)
        .collect(toList()); // 100, 102, 205, 300
```
  
### 스트림 축소
- 스트림은 주어진 값 이하의 크기를 갖는 새 스트림을 반환하는 `limit(n)` 메서드를 지원
  - 주어진 값 이하라는 것은 `limit(10)`을 했는데 현재 스트림의 요소 개수가 이보다 작다면 10개보다 작은 애들만 뽑히니까 그런 거임
```java
List<Integer> nums = Arrays.asList(1, 2, 3, 4, 5, 6, 7, 8, 9, 10);
List<Integer> evenNumbers = nums.stream()
        .filter(n -> n % 2 == 0)
        .limit(3)
        .collect(toList()); // 2, 4, 6
```
  
### 요소 건너뛰기
- 스트림은 처음 n개 요소를 제외한 스트림을 반환하는 `skip(n)` 메서드도 지원
```java
List<Integer> nums = Arrays.asList(1, 2, 3, 4, 5, 6, 7, 8, 9, 10);
List<Integer> evenNumbers = nums.stream()
        .filter(n -> n % 2 == 0)
        .skip(2)
        .collect(toList()); // 6, 8, 10


// limit와 활용
List<Integer> nums = Arrays.asList(1, 2, 3, 4, 5, 6, 7, 8, 9, 10);
List<Integer> evenNumbers = nums.stream()
        .filter(n -> n % 2 == 0)
        .skip(2)
        .limit(2)
        .collect(toList()); // 6, 8
```

<br>  

## 3. 매핑
- 특정 객체에 매핑되는 특정 데이터를 선택하는 작업을 말한다.

### map
- 인자로 `Function(객체를 인자로 받아 객체를 반환)`을 받아 각 요소에 적용한 결과가 매핑 (일종의 변환)
- 즉 매핑결과들의 스트림이 반환된다
```java
List<String> words = Arrays.asList("Apple", "Banana", "Orange");
List<Integer> wordLengths = words.stream()
        .map(String::length)
        .collect(toList());
```

### flatMap (스트림 평면화)
- 인자로 `Function`을 받아 각 요소에 적용한 결과가 매핑되는 건 map과 같음
- 그러나 차이가 있다면 인자로 받는 `Function`의 반환타입이 `Stream`이다
- 즉 `flatMap`은 일종의 펼치는 메서드
```java
List<List<String>> namesList = Arrays.asList(
    Arrays.asList("Kim", "Lee"),
    Arrays.asList("Park", "Choi")
);

List<String> flatList = namesList.stream()
        .flatMap(List::stream) // 각 리스트를 스트림으로 펼침
        .collect(toList());
```
 
<br>  

## 4. 검색과 매칭
### 적어도 한 요소는 일치하는 게 있는가
- `isSent`같은 변수를 선언하고, 어떤 배열에 대해 외부 반복을 수행하며 조건을 만족하는 요소가 있다면 isSent값을 바꾸는 로직이 있다고 하자
- 스트림에서는 `anyMatch`라는 메서드를 제공하며, 인자로 받은 `Predicate`가 일치하는 요소가 하나라도 있다면 true를 반환한다.
- `boolean`을 반환하는 최종연산이다
```java
boolean isSent = false;
if (messages.stream().anyMatch(Message::isSent)) {
    isSent = true;
)
```

### 모든 요소와 일치하는가
- 스트림은 `allMatch` 메서드를 통해 인자로 받은 `Predicate`가 모든 요소에 대해 true인지 검사 가능하다
```java
boolean isAllSent = messages.stream().allMatch(Message::isSent)
```
- `nonMatch`메서드는 `allMatch`의 반대 역할이다 (즉 모두 일치하지 않는가)

> 참고로 anyMatch, allMatch, noneMatch는 쇼트서킷 기법을 활용한다.
> && 연산이 앞에서 false이면 뒤에를 계산하지 않는 것처럼, 더 이상 판별하지 않아도 되면 멈춘다는 것임

### 요소 검색
- 스트림의 `findAny` 메서드는 현재 스트림에서 임의의 요소 하나를 `Optional`로 감싸서 반환
- 애도 쇼트서킷을 활용
- 첫 번째 요소를 찾으려면 `fintFirst`를 쓸 수 있다.
- 병렬 실행에서는 순서 보장이 힘드니 이때 보통 제약이 적은 `findAny`를 씀
