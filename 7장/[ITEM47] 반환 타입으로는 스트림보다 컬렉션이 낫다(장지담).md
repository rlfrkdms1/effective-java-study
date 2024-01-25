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


---

- 메서드가 스트림에서만 쓰인다 : 스트림 반환
- 메서드가 반복문에서만 쓰인다 : Iterable 반환
- 공개 API를 작성하는 상황이고, 정확히 모른다면 ..?

# Collection을 반환하자
- Collection 인터페이스는 Iterable의 하위 타입이고 stream 메서드도 제공한다.
- 원소 시퀸스를 반환하는 공개 API에서는 Collection이나 그 하위 타입을 쓰자.
- Arrays도 `Arrays.asList`와 `Stream.of`로 반복과 스트림을 지원할 수 있다.
- 



