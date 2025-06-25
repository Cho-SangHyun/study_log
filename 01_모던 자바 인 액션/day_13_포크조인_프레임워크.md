## 포크/조인 프레임워크
- 병렬스트림이 내부적으로 사용하는 "스레드풀"
- 병렬화할 수 있는 작업을 재귀적으로 작은 작업(=서브태스크)들로 분할한 다음, 서브태스크들의 결과를 합쳐 전체 결과를 만들도록 설계됨 (분할정복 알고리즘)
- "서브태스크를 워커 스레드들(스레드풀을 구성하는 스레드들)에 분산 할당하도록" `ExecutorService` 인터페이스를 구현하여 만들어져있다.

<br>  

### RecursiveTask<R>
- `ForkJoinPool`에서 수행되는 태스크(작업)의 형태 중 하나
- `R`은 병렬화된 태스크가 생성하는 결과 형식. 마땅한 결과 형식이 없으면 `RecursiveAction`을 사용
- `compute`라는 추상메서드를 가짐
  - 주어진 태스크를 서브태스크로 분할하는 로직과 더 이상 분할이 불가할 때 개별 서브태스크의 결과를 생산할 알고리즘을 이 메서드로 구현해야 함

1 ~ 10,000까지의 합을 포크/조인 프레임워크를 통해 분할정복 형태로 구할려면 다음과 같이 RecursiveTask를 만든다
```java
public class SampleCalculator extends RecursiveTask<Long> {
    private final long[] numbers;
    private final int left;
    private final int right;
    public static final long THRESHOLD = 10_000;

    public SampleCalculator(long[] numbers, int left, int right) {
        this.numbers = numbers;
        this.left = left;
        this.right = right;
    }

    @Override
    protected Long compute() {
        int currentRangeLength = right - left;

        if (currentRangeLength < THRESHOLD) {
            return calculateSum(left, right);
        }

        SampleCalculator leftTask = new SampleCalculator(numbers, left, left + currentRangeLength / 2);
        leftTask.fork(); // 다른 스레드에서 수행됨
        SampleCalculator rightTask = new SampleCalculator(numbers, left + currentRangeLength / 2, right);

        Long rightResult = rightTask.compute(); // 현재 스레드에서 수행됨
        Long leftResult  = leftTask.join();

        return leftResult + rightResult;
    }

    private long calculateSum(int left, int right) {
        long res = 0;
        for (int i = left; i < right; i++) {
            res += numbers[i];
        }
        return res;
    }
}
```

다음과 같이 포크조인풀에서 invoke를 호출해 해당 태스크를 실행시킨다.  
```java
long[] numbers = LongStream.rangeClosed(1, 10000).toArray();
ForkJoinTask<Long> task = new SampleCalculator(numbers, 0, numbers.length);
System.out.println(new ForkJoinPool().invoke(task));
```

<br>  

### 포크/조인 프레임워크 제대로 쓰기
1. join호출 시, 예를 들어 A태스크에서 B태스크를 만들고 B.join을 하면 B가 끝날 때까지 A가 블록됨. 따라서 A, B에서 모두 서브태스크가 시작된 뒤 join을 호출할 것
2. RecursiveTask 내에서는 ForkJoinPool의 invoke를 호출하지 말 것.
    - 대신 RecursiveTask에서 compute나 fork를 직접 호출 가능
    - 순차 코드에서 병렬 계산을 시작할 때만 invoke를 사용  
3. 서브태스크(얘도 RecursiveTask)에서 fork를 호출해서 포킹된 서브태스크를 다른 스레드에서 처리하도록 할 수 있으나, 포킹된 서브태스크를 모두 다른 스레드에서 처리하는 것보다 하나는 compute를 호출해서 기존 스레드에서 처리하는게 더 효율적임(스레드를 재사용하니까)
4. 멀티코어에 포크/조인 프레임워크를 쓰는게 순차처리보다 항상 빠르진 않음
    - 예를 들면 당연히 일단 서브태스크 처리시간이 새로운 태스크를 포킹하는 시간보다 길어야 의미가 있다

<br>  

### Work Stealing
- 말 그대로 작업을 훔치는 것
- 포크조인 프레임워크를 구성하는 스레드(워커 스레드)들이, 자기 일 끝내고 본인들이 담당하는 워크큐에 처리할 다른 일들이 없으면 다른 스레드의 워크큐 꼬리에 있는 거를 훔쳐오는 것
- 따라서 프로세서 수 이상으로 태스크가 존재해도 스레드 간 작업부하를 비슷한 수준으로 유지 가능
