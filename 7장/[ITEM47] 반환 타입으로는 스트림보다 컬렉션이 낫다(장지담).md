# Stream vs Iterable

![image](https://github.com/rlfrkdms1/effective-java-study/assets/96513365/d9bd69b5-cc77-4ffb-acf2-8a073758931a)

- 스트림으로 for-each를 사용할 수 없다. API가 스트림만 반환하면 스트림을 for-each로 반복하고 싶은 프로그래머들이 불만이다.
- 사실 스트림은 Iterable 인터페이스가 정의한 추상 메서드를 전부 포함하고 그대로 동작한다. 다만 Iterable을 확장하지 않을 뿐이다.

```java
        for (Integer num : (Iterable<Integer>) stream::iterator){
            System.out.println(num);
        }
```
- 메서드 참조와 형변환을 통해 사용할 수 있지만 별로다.

```java
    public static <E> Iterable<E> iterableOf(Stream<E> stream) {
        return stream::iterator;
    }
```
- 어댑터를 사용할 수 있다.

```java
    public static <E> Stream<E> streamOf(Iterable<E> iterable) {
        return StreamSupport.stream(iterable.spliterator(), false);
    }
```
- Iterable은 stream처럼 사용할 수 없다. 
- 반대로 Iterable을 스트림으로 변환하는 어댑터도 만들 수 있다.
- 하지만 어댑터는 클라이언트 코드를 지저분하게 하며 성능도 떨어진다.


---

- 메서드가 스트림에서만 쓰인다 : 스트림 반환
- 메서드가 반복문에서만 쓰인다 : Iterable 반환
- 공개 API를 작성하는 상황이고, 정확히 모른다면 ..?

# Collection을 반환하자
- Collection 인터페이스는 Iterable의 하위 타입이고 stream 메서드도 제공한다.
- 원소 시퀸스를 반환하는 공개 API에서는 Collection이나 그 하위 타입을 쓰자.
- Arrays도 `Arrays.asList`와 `Stream.of`로 반복과 스트림을 지원할 수 있다.

### 전용 컬렉션
- 반환 전부터 이미 컬렉션에 원소를 담아서 관리하고 있었거나, 컬렉션을 하나 더 만들어도 될 정도로 원소 개수가 적다면 ArrayList와 같은 표준 컬렉션에 담아 반환한다. 
- 반환할 시퀀스 크기가 크지만 간결하게 할 수 있다면 전용 컬렉션을 구현할 수 있다.
```java
public class PowerSet {
    // 코드 47-5 입력 집합의 멱집합을 전용 컬렉션에 담아 반환한다. (287쪽)
    public static final <E> Collection<Set<E>> of(Set<E> s) {
        List<E> src = new ArrayList<>(s);
        if (src.size() > 30)
            throw new IllegalArgumentException(
                "집합에 원소가 너무 많습니다(최대 30개).: " + s);
        return new AbstractList<Set<E>>() {
            @Override public int size() {
                // 멱집합의 크기는 2를 원래 집합의 원소 수만큼 거듭제곱 것과 같다.
                return 1 << src.size();
            }

            @Override public boolean contains(Object o) {
                return o instanceof Set && src.containsAll((Set)o);
            }

            @Override public Set<E> get(int index) {
                Set<E> result = new HashSet<>();
                for (int i = 0; index != 0; i++, index >>= 1)
                    if ((index & 1) == 1)
                        result.add(src.get(i));
                return result;
            }
        };
    }
```
- 원소 n개인 집합에 대해 2^n개의 원소를 갖는 멱집합은 크기가 큰 시퀀스다
- 각 원소를 포함하는지 여부를 비트 벡터로 나타내는 전용 컬렉션을 AbstractList로 구현한다.
- AbstractCollection을 사용해 Collection 구현체를 구현할 때는 Iterable용 메서드 외에 contains와 size만 구현하면 된다. 

---
> 가능하면 컬렉션을 반환하자. 이 때 전용 컬렉션 구현을 고려할 수 있다. 

> 컬렉션을 반환할 수 없으면 Iterable, Stream중 자연스러운 것을 반환하자



