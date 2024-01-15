# 제네릭 메서드
- 메서드도 제네릭으로 만들 수 있다.
- 매개변수화 타입(`List<String>`)을 받는 정적 유틸리티 메서드는 보통 제네릭이다.
- Collections의 binarySearch같은 알고리즘 메서드는 모두 제네릭이다.
```java
    public static <T> int binarySearch(List<? extends Comparable<? super T>> list, T key) {
        if (list instanceof RandomAccess || list.size()<BINARYSEARCH_THRESHOLD)
            return Collections.indexedBinarySearch(list, key);
        else
            return Collections.iteratorBinarySearch(list, key);
    }
```

제네릭이 아닌 로 타입을 활용하는 메서드를 보자.
```java
    public static Set union(Set s1, Set s2) {
        Set result = new HashSet();
        result.addAll(s2);
        return result;
    }
```
![image](https://github.com/rlfrkdms1/effective-java-study/assets/96513365/16b98408-eea1-4d5f-b07d-0cd22888b045)

- 컴파일은 가능하지만 로 타입을 사용해서 경고가 발생한다.

```java
    public static <E> Set<E> union(Set<E> s1, Set<E> s2) {
        Set<E> result = new HashSet<>(s1);
        result.addAll(s2);
        return result;
    }
```
- 제네릭으로 수정하면 타입 안전하다.
- 메서드의 제한자(`public`) 반환 타입(`Set<E>`) 사이에 타입 매개변수(`<E>`)를 선언한다

---

# 제네릭 싱글턴 팩터리
```java
public class GenericFactoryMethod {
public static final Set EMPTY_SET = new HashSet();

    public static final <T> Set<T> emptySet() {
        return (Set<T>) EMPTY_SET;
    }
}
출처: https://jake-seo-dev.tistory.com/13 [제이크서 위키 블로그:티스토리]
```
- 제네릭 싱글턴 팩터리는 제네릭으로 타입 설정 가능한 인스턴스(`EMPTY_SET`)를 만들어두고, 반환 시에 제네릭으로 받은 타입(`T`)을 이용해 타입을 결정(`Set<T>`)하는 것이다.
- 객체를 싱글턴으로 관리하면서도 제네릭 싱글턴 팩터리 메서드를 호출할 때마다 원하는 매개변수화 타입으로 캐스팅 해서 반환받을 수 있기 때문에 유연하다.
```java
    public static <T> Comparator<T> reverseOrder() {
        return (Comparator<T>) ReverseComparator.REVERSE_ORDER;
    }

    /**
     * @serial include
     */
    private static class ReverseComparator
        implements Comparator<Comparable<Object>>, Serializable {

        @java.io.Serial
        private static final long serialVersionUID = 7207038068494060240L;

        static final ReverseComparator REVERSE_ORDER
            = new ReverseComparator();

```
- Collections.reverseOrder 메서드는 내부 클래스 ReverseComparator의 인스턴스를 생성해놓고(REVERSE_ORDER), `Comparator<T>`로 캐스팅해서 반환한다.  
```java
    public static final Set EMPTY_SET = new EmptySet<>();

    /**
     * Returns an empty set (immutable).  This set is serializable.
     * Unlike the like-named field, this method is parameterized.
     *
     * <p>This example illustrates the type-safe way to obtain an empty set:
     * <pre>
     *     Set&lt;String&gt; s = Collections.emptySet();
     * </pre>
     * @implNote Implementations of this method need not create a separate
     * {@code Set} object for each call.  Using this method is likely to have
     * comparable cost to using the like-named field.  (Unlike this method, the
     * field does not provide type safety.)
     *
     * @param  <T> the class of the objects in the set
     * @return the empty set
     *
     * @see #EMPTY_SET
     * @since 1.5
     */
    @SuppressWarnings("unchecked")
    public static final <T> Set<T> emptySet() {
        return (Set<T>) EMPTY_SET;
    }
```
- `Collections.emptySet()` 도 인스턴스를 만들어두고 `Set<T>`로 캐스팅해서 반환한다.
```java
    public static void main(String[] args) {
        Comparator<Robot> robotComparator = Collections.reverseOrder();
        Comparator<Integer> integerComparator = Collections.reverseOrder();
        Comparator<String> stringComparator = Collections.reverseOrder();
    }
```
- 제네릭 싱글턴 팩터리를 통해 원하는 매개변수화 타입의 객체를 얻는다.
- ReverseComparator 객체를 싱글턴으로 관리하면서 유연함도 챙길 수 있다
![image](https://github.com/rlfrkdms1/effective-java-study/assets/96513365/8d76f44e-4e75-4ea8-89af-49cf0939f99d)

# 재귀적 타입 한정 
- 자기 자신이 들어간 표현식을 사용해 타입 매개변수의 허용 범위를 한정한다. 
- 주로 Comparable과 함께 쓰인다
- 클래스가 `Comparable<T>`를 구현한다는 것은 해당 클래스의 객체와 T 객체를 비교할 수 있다는 뜻이다. 예를 들어 Robot은 Robot과 비교할 수 있기 때문에 `Comparable<Robot>`을 구현한다.
- Comparable을 구현한 원소의 컬렉션을 입력받는 메서드들은 주로 원소들을 정렬, 검색, 최소&최대값 구하기 등을 한다. 즉 컬렉션의 원소들은 모두 상호 비교가 가능해야 한다.
```java
    public static <E extends Comparable<E>> E sort(Collection<E> collection){
        ...
    }
```
- 재귀적 타입 한정을 통해 `Collection<E>`의 모든 원소들은 상호 비교가 가능함을 보장한다.
- E는 `Comparable<E>`를 상속한 타입이다. 즉 E는 자기 자신인 E와 비교가 가능하다. 
