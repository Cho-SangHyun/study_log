### Trader 클래스
```java
public class Trader {
    private final String name;
    private final String city;

    public Trader(String name, String city) {
        this.name = name;
        this.city = city;
    }

    public String getName() {
        return name;
    }

    public String getCity() {
        return city;
    }

    @Override
    public String toString() {
        return "Trader{" +
                "name='" + name + '\'' +
                ", city='" + city + '\'' +
                '}';
    }
}
```

### Transaction 클래스
```java
public class Transaction {
    private final Trader trader;
    private final int year;
    private final int value;

    public Transaction(Trader trader, int year, int value) {
        this.trader = trader;
        this.year = year;
        this.value = value;
    }

    public Trader getTrader() {
        return trader;
    }

    public int getYear() {
        return year;
    }

    public int getValue() {
        return value;
    }

    @Override
    public String toString() {
        return "Transaction{" +
                "trader=" + trader +
                ", year=" + year +
                ", value=" + value +
                '}';
    }
}
```

### 연습에 사용될 데이터
```java
Trader raoul = new Trader("Raoul", "Cambridge");
Trader mario = new Trader("Mario", "Milan");
Trader alan = new Trader("Alan", "Cambridge");
Trader brian = new Trader("Brian", "Cambridge");
        
List<Transaction> transactions = Arrays.asList(
    new Transaction(brian, 2011, 300),
    new Transaction(raoul, 2012, 1000),
    new Transaction(raoul, 2011, 400),
    new Transaction(mario, 2012, 710),
    new Transaction(mario, 2012, 700),
    new Transaction(alan, 2012, 950)
);
```

## 문제
### 1. 2011년에 일어난 모든 트랜잭션을 찾아 값을 오름차순 정렬
```java
List<Transaction> transactions2011 = transactions.stream()
        .filter(t -> t.getYear() == 2011)
        .sorted(comparing(Transaction::getValue))
        .collect(toList());
```

### 2. 거래자가 근무하는 모든 도시를 중복없이 나열
```java
List<String> traderCities = transactions.stream()
        .map(t -> t.getTrader().getCity())
        .distinct()
        .collect(toList());
```

### 3. 케임브리지에서 근무하는 모든 거래자를 찾아 이름순 정렬
```java
List<Trader> tradersInCambridge = transactions.stream()
        .filter(t -> "Cambridge".equals(t.getTrader().getCity()))
        .map(Transaction::getTrader)
        .distinct()
        .sorted(comparing(Trader::getName))
        .collect(toList());
```

### 4. 모든 거래자 이름을 알파벳순으로 정렬해 반환
```java
List<String> traderNames = transactions.stream()
        .map(t -> t.getTrader().getName())
        .distinct()
        .sorted(comparing(name -> name))
        .collect(toList());
```

### 5. 밀라노에 거래자가 있는가?
```java
boolean isExistMilan = transactions.stream()
        .map(t -> "Milan".equals(t.getTrader().getCity()))
        .reduce(false, (a, b) -> a || b);

// 정답
boolean isExistMilan = transactions.stream()
        .anyMatch(t -> "Milan".equals(t.getTrader().getCity()))
```

### 6. 전체 트랜잭션 중 최댓값은?
```java
Optional<Integer> maxTransactionValue = transactions.stream()
        .map(Transaction::getValue)
        .reduce(Integer::max);
```

