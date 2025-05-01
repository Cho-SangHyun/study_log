### 생성자 참조
- `ClassName::new`처럼 활용한다
- 생성자 역시 파라미터가 없는 생성자, 파라미터가 있는 생성자 등이 있을 텐데 각각 다음 함수형 인터페이스에 대응한다고 보면 된다
  - 파라미터 없는 생성자 : `Supplier`
  - 파라미터 있는 생성자 : `Function`

<br> 

예를 들어, `Function<Integer, Apple>`은 Integer타입 객체를 파라미터로 받아 Apple타입을 반환하는 함수형 디스크립터를 묘사한다.  
이때 Integer타입의 파라미터를 받는 Apple클래스의 생성자는 `Interger -> Apple`로 묘사할 수 있으므로 `Apple::new`를 `Function<Integer, Apple>`이 쓰이는 컨텍스트에 사용 가능하다
