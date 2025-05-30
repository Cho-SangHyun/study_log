## 람다에서의 지역변수
- 람다 표현식도 파라미터로 넘겨지는 변수가 아닌 외부에서 정의된 변수를 활용 가능 (`람다 캡처링`이라 부름)
- 인스턴스 변수, static 변수에는 별도 제약없이 접근 가능
- 지역변수에 접근하려면 이들은 명시적으로 final로 선언되어 있거나 final로 선언된 것처럼 사용되어야 함
  - final로 선언된 것처럼 사용되어야 한다는 것 = 변수가 초기화된 후 한 번도 변경이 일어나지 않아야 한다는 것  
- 즉 람다 표현식은 한 번만 할당할 수 있는 지역변수만 캡쳐 가능

<br>  

즉 이런 거 안 된다.  
```java
int num = 123;
Runnable r = () -> System.out.println(num);
num = 456;
```

<br>  

근데 왜일까? 이유는 `지역변수는 스택에 만들어지기 때문`이다.
- 지역변수를 관리하는 스레드와 람다를 실행하는 스레드가 달라지면, 변수를 관리한 스레드가 사라졌으나 람다를 실행하는 스레드에서는 해당 변수(즉 더 이상 사용하지 않는 스택 공간)에 계속 접근하려 할 수 있게 됨
- 그래서 람다가 지역변수를 참조할 땐 지역변수가 있는 스택 공간 자체에 접근하는게 아니라, 그 지역 변수를 복사한 값을 따로 두고 있다가 걔를 참조함
- 근데 그 지역변수의 값이 나중에 바뀌면, 람다가 복사해 들고 있는 값과 달라지는 상황이 생김. 즉 `일관성이 깨지는 문제`가 발생할 수 있다
- 그래서 람다가 참조하는 지역변수는 '절대 바뀌지 않는 것'만 허용
- 인스턴스 변수, static 변수는 힙 영역에 만들어져서 괜춘

<br>  

## 메서드 참조
- 메서드 참조 : 특정 메서드만을 호출하는 람다의 축약형
- 이를 활용하면 기존 메서드 구현으로 람다 표현식을 만들 수 있음
- 작성된 메서드 참조를 컴파일러가 자동으로 해당 위치의 컨텍스트에 맞는 람다 표현식으로 변환해주는 원리
  - 타입 검사, 추론 때와 마찬가지로 그 컨텍스트에서 기대되는 함수형 인터페이스가 있을 것임(대상 형식!)
  - 그 놈의 추상메서드의 함수 디스크립터에 맞게 변환해준다고 보면 됨
 
### 예제 (내가 직접 활용한)
다음과 같은 `List<HashMap<String, Object>>` 타입의 변수가 있었다.  

```java
// 리스트의 각 아이템들(HashMap<String, Object>)은 "AUDIT_NBR", "PNOT_SBJC_NBR"이란 키를 무조건 가진다
List<HashMap<String, Object>> pointedSubjects = pointedSubjectsService.getAllPointedSubjects();
```

pointedSubjects에서 "AUDIT_NBR", "PNOT_SBJC_NBR" 키값들의 쌍들만 추출하고 싶으면 다음과 같이 써줄 수 있었다.

```java
List<HashMap<String, Object>> auditNbrAndPnotSbjcNbrPairs = pointedSubjects.stream()
    .map(item -> {
        HashMap<String, Object> pair = new HashMap<>();
        pair.put("AUDIT_NBR", item.getOrDefault("AUDIT_NBR", ""));
        pair.put("PNOT_SBJC_NBR", item.getOrDefault("PNOT_SBJC_NBR", 0));
        return pair;
    })
    .collect(Collectors.toList());
```

그러나 stream의 map은 `Function<T, R>`이라는 함수형 인터페이스를 파라미터로 받는다.  
즉 다음과 같이 `HashMap<String, Object>`를 파라미터로 받고 `HashMap<String, Object>`를 리턴하는 메서드를 작성한 뒤 메서드 참조를 사용해줄 수 있었다.

```java
    List<HashMap<String, Object>> auditNbrAndPnotSbjcNbrPairs = pointedSubjects.stream()
        .map(this::extractAuditNbrAndPnotSbjcNbr)
        .collect(Collectors.toList());


public HashMap<String, Object> extractAuditNbrAndPnotSbjcNbr(HashMap<String, Object> data) {
    HashMap<String, Object> pair = new HashMap<>();
    pair.put("AUDIT_NBR", item.getOrDefault("AUDIT_NBR", ""));
    pair.put("PNOT_SBJC_NBR", item.getOrDefault("PNOT_SBJC_NBR", 0));
    return pair;
}
```
