# 자바8 이전
특정 조건에서 값을 반환할 수 없을 때 

## 예외 던지기
진짜 예외적인 상황에서만 사용해야한다. 
예외를 생성할 때 스택 추적 전체를 캡처 -> 비용이 만만치 않다. 

## null 반환
null을 반환할 수 있는 메서드를 호출할 때는 별도의 null 처리 코드를 추가해야한다. 이를 무시하면 이상한 곳에서 NPE가 발생할 수 있다. 

# `Optional<T>`
따라서 이를 해결하기 위해 `Optional<T>`라는 것이 생겼다. 이는 null이 아닌 T 타입 참조를 하나 담거나, 혹은 아무것도 담지 않을 수 있다. 

원소를 최대 1개 가질 수 있는 **불변** 컬렉션이다. 

T를 반환할 때 `Optional<T>`를 반환하면 된다. 이는 예외를 던지는 메서드보다 유연하고 사용하기 쉬우며, null을 반환하는 메서드보다 오류 가능성이 적다. 

```java
    public static <E extends Comparable<E>> E max(Collection<E> c) {
        if (c.isEmpty())
            throw new IllegalArgumentException("빈 컬렉션");

        E result = null;
        for (E e : c)
            if (result == null || e.compareTo(result) > 0)
                result = Objects.requireNonNull(e);

        return result;
    }
```
이 메서드에 빈 컬렉션을 건네면 if문을 통해 `IllegalArgumentException` 예외를 던진다. 따라서 `Optional<E>`를 사용하도록 수정해보자. 

```java
    public static <E extends Comparable<E>>
    Optional<E> max(Collection<E> c) {
        if (c.isEmpty())
            return Optional.empty();

        E result = null;
        for (E e : c)
            if (result == null || e.compareTo(result) > 0)
                result = Objects.requireNonNull(e);

        return Optional.of(result);
    }
```
따라서 비었을 경우 `Optional.empty()`를 통해 빈 옵셔널을 반환하고 비지 않았을 때는 `Optional.of(result);`을 통해 값을 넣어 반환하면 된다. 

이때 `Optional.of(null);`, 즉, `Optional.of()`에 인수로 null을 넣으면 NPE를 던지니 주의해야한다. null 값도 허용하는 옵셔널을 만드려면 `Optional.ofNullable(value)`를 사용하면 된다. 

옵셔널을 반환하는 메서드에서는 절대 null을 반환하지 말자. 

스트림의 종단 메서드는 Optional을 반환하는 경우가 많다. 

```java
    public static <E extends Comparable<E>>
    Optional<E> max(Collection<E> c) {
        return c.stream().max(Comparator.naturalOrder());
    }
```
이 경우 `max()`메서드는 Optional을 반환하는데, 아래의 `max()`메서드의 형태를 보면 알 수 있다. 
```java
    Optional<T> max(Comparator<? super T> comparator);
```

# 옵셔널 사용 기준

> Optional은 검사 예외와 취지가 비슷하다. 

즉, 반환 값이 없을 수도 있음을 API 사용자에게 명확히 알려준다. 

## 활용
클라이언트가 값을 받지 못했을 때 취할 행동을 선택할 수 있다. 

### 기본 값을 정해둘 수 있다. 

```java
String lastWordInLexicon = max(words).orElse("단어 없음...");
```
위의 경우에서는 max의 words가 값이 없다면 "단어 없음..."을 출력한다. 또는 아래와 같으며 `orElse()`문에 들어오는 인자는 `max()`메서드의 인자와 같은 타입이어야 한다. 

```java
        List<Integer> numbers = List.of(1, 2, 3, 4, 5);
        numbers.stream().max(Comparator.naturalOrder()).orElse(100);
```

### 원하는 예외를 던질 수 있다. 

```java
public enum Cheese {
    STILTON, CHEDDAR, MOZZARELLA;

    public static Cheese of(String cheeseName) {
        return Arrays.stream(Cheese.values())
                .filter(c -> c.name().equals(cheeseName))
                .findAny()
                .orElseThrow(IllegalArgumentException::new);
    }
}
```

위와 같이 원하는 Cheese가 없을 경우 지정한 예외인 `IllegalArgumentException`를 던질 수 있다. 

### 항상 값이 채워져 있다고 가정한다. 

```java
        List<Integer> numbers = List.of(1, 2, 3, 4, 5);
        Integer maxNumber = numbers.stream().max(Comparator.naturalOrder()).get();
```
항상 값이 채워져 있다고 가정한다면 `get()`을 사용해 바로 값을 꺼내 사용할 수 있다. 하지만 잘못 판단한 것이라면 `NoSuchElementException`이 발생한다. 

## `Supplier<T>`
기본값을 설정하는 비용이 커 부담이 된다면 `Supplier<T>`를 인수로 받는 `orElseGet`을 사용할 수 있다. 이를 사용하면 `Supplier<T>`를 사용해 생성하므로 초기 설정 비용을 낮출 수 있기 때문이다. 


## `isPresent`
옵셔널이 채워져있으면 true, 비어있으면 false를 반환한다. 

다음의 예시는 부모 프로세스의 PID를 출력하거나, 부모가 없다면 "N/A"를 출력하는 코드다. 

```java
        ProcessHandle ph = ProcessHandle.current();

        Optional<ProcessHandle> parentProcess = ph.parent();
        System.out.println("부모 PID: " + (parentProcess.isPresent() ?
                String.valueOf(parentProcess.get().pid()) : "N/A"));
```
하지만 이는 `Optional`의 `map()`을 사용해 아래와 같이 작성할 수 있다. 

```java
        System.out.println("부모 PID: " +
                ph.parent().map(h -> String.valueOf(h.pid())).orElse("N/A"));
```
아래와 같이 구현할 수도 있다. 

```java
        Stream<Optional<String>> streamOfOptional = Stream.of(Optional.of("hi"));
        streamOfOptional.filter(Optional::isPresent).map(Optional::get);
```
Optional을 stream으로 변환할수도 있다. 

```java
streamOfOptional.flatMap(Optional::stream);
```
옵셔널에 값이 있으면 값을 담은 스트림으로, 없다면 빈 스트림으로 변환한다. 

## 사용하지 말아야할 때
항상 옵셔널을 사용하면 득이될까? 아니다. 

> 컬렉션, 스트림, 배열, 옵셔널 같은 컨테이너 타입은 옵셔널로 감싸면 안된다. 

빈 `Optional<List<T>>`를 반환하기 보다는 빈 `List<T>`를 반환하는게 좋다. 후자에선 클라이언트가 옵셔널 처리를 안해도 되기 때문이다. 

## 사용할 때
그렇다면 어떤경우에 사용할까? 

> 결과가 없을 수 있으며, 클라이언트가 이 상황을 특별하게 처리해야할 때

하지만 이렇다로 하더라도 `Optional`을 새로 할당해 초기화 하고 메서드를 호출해 값을 꺼내야하니 대가가 따른다. 따라서 성능이 중요하다면 옵셔널이 맞지 ㅇ낳을 수도 있다. 

박싱된 기본 타입을 담는 옵셔널은 기본 타입 자체보다 무거울 수 밖에 없다. 값을 두번이나 감싸기 때문이다. (박싱으로 한번, 옵셔널로 한번)

따라서 기본 타입 전용 옵셔널 클래스인 `OptionalInt, OptionalLong, OptionalDouble`이 존재한다. 

> 박싱된 기본 타입을 담은 옵셔널을 반환하는 일은 없도록하자. 

## 주의

옵셔널을 맵의 값으로 사용하면 절대 안된다. 
맵 안에 키가 없다는 사실을 나타내는 방법이 두가지나 되기 때문이다. 
- 키가 없는 경우
- 키는 있느나 값이 없는 경우(빈 옵셔널)

> 옵셔널을 컬렉션의 키, 값, 원소나 배열의 원소로 사용하는게 적절한 상황은 거의 없다. 

옵셔널을 인스턴스 필드에 저장해두는 것은 좋지 않은 상황이다. 적절할 때도 있는데 필수 필드가 아니며 기본 타입이라 값이 없음을 나타낼 방법이 마땅치 않을 때다. 


