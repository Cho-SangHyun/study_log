## 리듀싱
- 리듀싱 연산 : 모든 스트림 연산을 처리해 값으로 도출하는 연산
   - ex 1) 매뉴의 모든 칼로리의 합계를 구하라
   - ex 2) 메뉴에서 가장 높은 칼로리를 구하라
 
<br>  

기본적으로 외부 반복을 사용하던 시절엔 다음처럼 했었다.
```java
// ex 1) 매뉴의 모든 칼로리의 합계를 구하라
int sum = 0;
for (Dish dish : menus) {
    sum += dish.getCalories();
}

// ex 2) 메뉴에서 가장 높은 칼로리를 구하라
int maxCalories = -1;
for (Dish dish : menus) {
    maxCalories = Math.max(maxCalories, dish.getCalories());
}
```
<br>  

이를 `reduce`를 사용하면 다음처럼 나타낼 수 있다.
```java
// ex 1) 매뉴의 모든 칼로리의 합계를 구하라
int sum = menus.stream().reduce(0, (a, b) -> a + b);

// ex 2) 메뉴에서 가장 높은 칼로리를 구하라
int maxCalories = menus.stream().reduce(01, (a, b) -> Math.max(a, b));
```
<br>  

`reduce`의 인자로 초깃값 0과 두 요소를 조합해 새 값을 만드는 `BinaryOperator<T>`를 준 모습.  
초깃값을 받지 않아도 되는 오버로드된 `reduce`도 존재하며 이 놈은 `Optional`을 리턴한다. 빈 스트림에 대한 `reduce`는 `null`도 표현할 수 있어야 하기 때문.  
초기값을 받지 않는 reduce를 사용하면 다음처럼 나타낼 수 있다.
```java
// ex 1) 매뉴의 모든 칼로리의 합계를 구하라
Optional<Integer> sum = menus.stream().reduce((a, b) -> a + b);

// ex 2) 메뉴에서 가장 높은 칼로리를 구하라
Optional<Integer> maxCalories = menus.stream().reduce((a, b) -> Math.max(a, b));
```
<br>  

Integer는 sum, max라는 정적 메서드를 제공하므로 다음처럼 바꿀 수도 있다.
```java
// ex 1) 매뉴의 모든 칼로리의 합계를 구하라
Optional<Integer> sum = menus.stream().reduce(Integer::sum);

// ex 2) 메뉴에서 가장 높은 칼로리를 구하라
Optional<Integer> maxCalories = menus.stream().reduce(Integer::max);
```
<br>  

### 스트림 연산 - stateless vs stateful
- 스트림에서 제공되는 연산(`filter`, `map`, `distinct`, `skip`, `reduce` 등)은 스트림의 내부 상태를 고려해야 한다.
- `map`, `filter` 등은 입력 스트림에서 각 요소를 받아 결과를 출력 스트림으로 보내는 연산
  - 따라서 인자로 받는 람다나 메서드 참조가 내부적인 상태가 없다면 이들은 보통 내부 상태가 없는 stateless 연산
- `reduce` 등의 연산은 결과를 누적할 내부 상태가 필요. 이 값은 `int`나 `long`같은 값이니, 스트림 요소에 상관없이 용량야 한계가 있음
  - `distinct` 연산도 중복을 제거하려면 과거의 처리 이력을 내부적으로 알고 있어야 함. 이런 연산들을 `stateful` 연산이라 부름
