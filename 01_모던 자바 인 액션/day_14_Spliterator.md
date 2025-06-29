## Spliterator
- 자동으로 스트림을 분할하는 기법
- Splitable Iterator, 분할 가능한 반복자라는 의미를 가짐
- `Iterator`처럼 소스의 요소들을 탐색하는 기능을 제공한다는 점은 같으나 병렬 처리에 특화된 반복자
- 자바 8은 컬렉션 프레임워크에 포함된 모든 자료구조(`List`, `Set` 등)에 사용할 수 있는 디폴트 `Spliterator` 구현을 제공
  - 컬렉션은 `spliterator`라는 메서드를 제공하며, 이는 해당 컬렉션에서 사용가능한 디폴트 `Spliterator`를 제공

```java
List<String> names = Arrays.asList(
        "Alice", "Bob", "Charlie", "David", "Eve",
        "Frank", "Grace", "Hannah", "Ivan", "Judy"
);

// 예시
Spliterator<String> spliterator = names.spliterator();
```
<br>  

### Spliterator 인터페이스 (디폴트 메서드 제외)
```java
public interface Spliterator<T> {
    boolean tryAdvance(Consumer<? super T> action);

    Spliterator<T> trySplit();

    long estimateSize();

    int characteristics();
}
```
  
참고 : T = 해당 `Spliterator`가 참조하는 스트림의 데이터 요소 타입


| 메서드                                                 | 설명                                                         |
| --------------------------------------------------- | ---------------------------------------------------------- |
| `boolean tryAdvance(Consumer<? super T> action)`    | 현재 요소를 소비하고, 탐색할 요소가 남아있으면 `true`, 없으면 `false` 리턴 |
| `Spliterator<T> trySplit()`                         | 이 Spliterator가 처리 중인 데이터의 일부를 잘라 새 `Spliterator`를 생성 (병렬 처리용 분할)                |
| `long estimateSize()`                               | 남은 요소 개수를 추정하여 반환 (즉 이 값은 정확하지 않을 수도)                                         |
| `int characteristics()`                             | 이 Spliterator의 특성을 비트 플래그로 반환 (ORDERED, SORTED 등)          |

<br>  

### 아니 그래서 얘로 뭐 한다는 거임
- 책에서 WordCounter 예제 들면서 이렇게 커스텀해서 병렬처리할 수 있다~ 고 하긴 하는데 와닿지가 않는다.
- 책에 있는 WordCounter 예제 들고와봤자 아무 의미없을 것 같아서 `Splitertor`가 뭔지는 내 방식대로 정리하기로 함.
- 일단 요점은. 스트림에서 `parallelStream` 호출해서 병렬스트림으로 만들면 이렇게 병렬쳐리를 한다는 것


1. `parallelStream()` → 내부적으로 `Spliterator`를 통해 그 스트림의 데이터를 분할
2. 분할된 작업은 Fork/Join 프레임워크 기반으로 병렬로 수행됨
3. 그래서 우리는 `parrallelStream`만 호출하면 Stream API가 이들을 추상화하여 자동으로 처리한다는 거

걍 좀 더 내가 파봐야겠다. 
