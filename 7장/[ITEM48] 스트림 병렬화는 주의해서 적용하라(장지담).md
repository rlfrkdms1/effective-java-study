# 스트림 병렬화를 주의해서 사용하라
- 병렬화는 여러 작업이 동시에 처리되는 것이다. Stream에서 `pararrel()`메서드로 사용한다. 
- 데이터 소스가 stream.iterate거나 중간 연산으로 limit을 쓰면 파이프라인 병렬화로 성능 개선을 기대할 수 없다.

### 스트림을 잘못 병렬화하면 성능이 나빠지고 결과가 잘못되고 예상치 못한 동작이 발생한다. 
- 스트림 파이프라인을 마구잡이로 병렬화하면 성능이 나빠질 수 있다
  - limit을 다룰 때 cpu 코어가 남는다면 원소를 몇개 더 처리하고 버린다. 원소 하나 처리에 엄청난 시간이 걸리는 경우 이는 큰 낭비다.
> 안전 실패 : 결과가 잘못되고 오작동함

- 안전 실패는 병렬화된 파이프라인이 사용하는 mappers, filters, 다른 함수 객체들이 오작동 할 때 벌어진다. Stream은 함수 객체에 대해 엄중한 규약을 정의해놨다. 결합법칙을 만족하고, 간섭받지 않고, 상태를 같지 않아야 한다.
  - 이러한 요구사항을 지키지 않아도 순차적으로 수행하면 성공할 수도 있지만, 병렬로 수행하면 실패한다.

### 병렬화는 최적화 수단이다
- 적용 전, 후의 성능을 비교하자

# 병렬화 효과
### 스트림의 소스가 ArrayList, HashSet, HashMap, ConcurrentHashMap, 배열, int범위, long범위 일 때 좋다
- 데이터를 원하는 크기로 정확하고 손쉽게 나눌 수 있다
  - 이 작업은 Spliterator 객체가 담당하며 Stream이나 Iterable의 spliterator 메서드로 얻는다
- 이웃한 원소의 참조들이 메모리에 연속해서 저장되어 있다.
- 참조들이 가리키는 실제 객체가 메모리에서 떨어져 있으면 참조 지역성이 나쁘다. 참조 지역성은 병렬화에서 중요한 요소며 기본 타입의 배열이 참조 지역성이 뛰어나다. (데이터 자체가 연속되어 저장된다)
### 병렬화에 적합한 종단 연산일 때 좋다
- 축소 : Stream의 reduce 메서드 중 하나, min, max, count, sum
- 조건에 맞으면 바로 반환하는 메서드 : anyMatch, allMatch, noneMatch
- Stream의 collect는 적절하지 않다.

### 파이프라인이 수행하는 작업이 병렬화에 드는 추가 비용을 상쇄해야 한다 
- `스트림 안의 원소 수 * 원소당 수행되는 코드 줄 수 > 수십만` 은 되어야 성능 향상을 기대할 수 있다. 

# 무작위 수들로 이루어진 스트림 병렬화
- ThreadLocalRandom , Random 보다 SplittableRandom을 사용하자. 병렬화하면 성능이 선형으로 증가한다.
- ThreadLocalRandom은 단일 스레드에서 쓰고자 만들어졌고, Random은 병렬 처리에서 최악이다. 

---

> 조건이 잘 갖춰지면 parallel 메서드 호출 하나로 거의 프로세서 코어 수에 비례하는 성능 향상을 만끽할 수 있다. 

```java
    static long pi(long n) {
        return LongStream.rangeClosed(2, n)
                .parallel()
                .mapToObj(BigInteger::valueOf)
                .filter(i -> i.isProbablePrime(50))
                .count();
    }
```


