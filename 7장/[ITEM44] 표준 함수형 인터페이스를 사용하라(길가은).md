# 함수형 인터페이스
람다를 지원하면서 현대에선 함수 객체를 매개변수로 받는 생성자와 메서드를 더 많이 만들어야한다. 이때 함수형 매개변수 타입을 올바르게 선택해야 한다. 

`LinkedHashMap`을 생각해보자. 이 클래스의 `removeEldestEntry`는 재정의하면 캐시로 사용할 수 있다. 

```java
    protected boolean removeEldestEntry(Map.Entry<K,V> eldest) {
        return false;
    }
```
아래의 put 메서드를 보면 마지막에 `afterNodeInsertion()` 메서드를 호출한다. `afterNodeInsertion` 메서드 내에서 `removeEldestEntry`를 호출한다. 


```java
    public V put(K key, V value) {
        return putVal(hash(key), key, value, false, true);
    }

    /**
     * Implements Map.put and related methods.
     *
     * @param hash hash for key
     * @param key the key
     * @param value the value to put
     * @param onlyIfAbsent if true, don't change existing value
     * @param evict if false, the table is in creation mode.
     * @return previous value, or null if none
     */
    final V putVal(int hash, K key, V value, boolean onlyIfAbsent,
                   boolean evict) {
        Node<K,V>[] tab; Node<K,V> p; int n, i;
        if ((tab = table) == null || (n = tab.length) == 0)
            n = (tab = resize()).length;
        if ((p = tab[i = (n - 1) & hash]) == null)
            tab[i] = newNode(hash, key, value, null);
        else {
            Node<K,V> e; K k;
            if (p.hash == hash &&
                ((k = p.key) == key || (key != null && key.equals(k))))
                e = p;
            else if (p instanceof TreeNode)
                e = ((TreeNode<K,V>)p).putTreeVal(this, tab, hash, key, value);
            else {
                for (int binCount = 0; ; ++binCount) {
                    if ((e = p.next) == null) {
                        p.next = newNode(hash, key, value, null);
                        if (binCount >= TREEIFY_THRESHOLD - 1) // -1 for 1st
                            treeifyBin(tab, hash);
                        break;
                    }
                    if (e.hash == hash &&
                        ((k = e.key) == key || (key != null && key.equals(k))))
                        break;
                    p = e;
                }
            }
            if (e != null) { // existing mapping for key
                V oldValue = e.value;
                if (!onlyIfAbsent || oldValue == null)
                    e.value = value;
                afterNodeAccess(e);
                return oldValue;
            }
        }
        ++modCount;
        if (++size > threshold)
            resize();
        afterNodeInsertion(evict);
        return null;
    }
```

```java
    void afterNodeInsertion(boolean evict) { // possibly remove eldest
        LinkedHashMap.Entry<K,V> first;
        if (evict && (first = head) != null && removeEldestEntry(first)) {
            K key = first.key;
            removeNode(hash(key), key, null, false, true);
        }
    }
```
따라서 `removeEldestEntry`를 다음ㄱ뫄 같이 재정의해 true를 반환하면 map에서 가장 오래된 원소를 제거 한다. 따라서 가장 최근 원소 100개를 유지한다.

```java
    protected boolean removeEldestEntry(Map.Entry<K,V> eldest) {
        return size() > 100;
    }
```

람다를 사용하면 더 잘 해낼 수 있다. 위 코드에서 함수 객체는 `Map.Entry<K,V>`를 인자로 받고 있다. 하지만 생성자에 넘기는 함수 객체는 이 맵의 인스턴스 메서드가 아니다. 팩터리나 생성자를 호출할 때는 맵의 인스턴스가 존재하지 않기 때문이다. 따라서 맵은 자기 자신도 함수 객체에 건네줘야한다. 따라서 아래와 같은 인터페이스를 만들어 사용할 수 있다. 

```java
@FunctionalInterface interface EldestEntryRemovalFunction<K,V> {
    boolean remove(Map<K, V> map, Map.Entry<K, V> eldest);
}
```

하지만 우리는 위의 인터페이스를 만들어 사용할 필요가 없다. 자바에 같은 모양으로 표준 함수형 인터페이스가 존재하기 때문이다. 

## 표준 함수형 인터페이스
> 필요한 용도에 맞는게 있다면, 직접 구현하지 말고 표준 함수형 인터페이스를 활용하라.

### 장점
- API가 다루는 개념의수가 줄어 익히기 더 쉬움
- 유용한 디폴트 메서드를 많이 제공하므로 다른 코드와의 상호 운용성이 크게 좋아짐

표준 함수형 인터페이스 43개 중 기본 인터페이스 6개에 대해 알아보자. 

| 인터페이스 | 함수 시그니처 | 의미 |  예 |
| ------- | ---------- | --- | -- |
| `UnaryOperator<T>` | `T apply(T t)` | 반환 값과 인수의 타입이 같은 함수, 인수 1개 | `String::toLowerCase` |
|`BinaryOperator<T>` | `T apply(T t1, T t2)` | 반환 값과 인수의 타입이 같은 함수, 인수 2개 | `BigInteger::add` |
| `Predicate<T>` | `boolean test(T t)` | 인수 하나를 받아 boolean을 반환하는 함수 | `Collection::isEmpty` |
| `Function<T, R>` | `R apply(T t)` | 인수와 반환 타입이 다른 함수 | `Arrays::asList` |
| `Supplier<T>` | `T get()` | 인수를 받지 않고 값을 반환(혹은 제공)하는 함수 | `Instant::now` |
| `Consumer<T>` | `void accept(T t)` | 인수 하나 받고 반환 값은 없는 함수 | `System.out::println` |

기본 인터페이스는 기본 타입인 `int, long, double`용으로 각 3개씩 변형이 생기며 이름도 앞에 기본 타입 이름을 붙여 지었다. 
Function 인터페이스에는 기본 타입을 반환하는 변형이 총 9개가 더 있다. 입력과 결과의 타입이 항상 다르므로 둘 다 기본 타입이면 접두어로 `SrcToResult`를 사용한다. 입력을 매개변수화한 변형은 접두어로 `ToResult`를 사용한다. 
기본 함수형 인터페이스 중 3개에는 인수를 2개씩 받는 변형이 있으며 총 9개의 변형이 존재한다.
BooleanSupplier인터페이스는 boolean을 반환하도록 한 Supplier의 변형이다. 

표준 함수형 인터페이스 대부분은 기본 타입만 지원한다. 그렇다고 기본 함수형 인터페이스에 박싱된 기본 타입을 넣어 사용하지는 말자. 계산량이 많을 때 성능이 처참히 느려질 수 있다. 

## 직접 작성할 때
표준 인터페이스 중 필요한 용도에 맞는게 없다면 직접 작성해야한다. 

`Comparator<T>` 인터페이스는 구조적으로 `ToIntBiFunctioin<T,U>`와 동일하지만 독자적인 인터페이스로 살아남아야 하는 이유가 있다. 

1. API에서 굉장히 자주 사용되는제, 지금의 이름이 그 용도를 아주 훌륭히 설명해준다. 
2. 구현하는 쪽에서 반드시 지켜야 할 규약을 담고 있다. 
3. 비교자들을 변환하고 조합해주는 유용한 디폴트 메서드들을 듬뿍 담고 있다. 

이 중 하나 이상을 만족한다면 전용 함수형 인터페이스를 구현해야하는 건 아닌지 진중히 고민해야한다. 

- 자주 쓰이며, 이름 자체가 용도를 명확히 설명해준다. 
- 반드시 따라야 하는 규약이 있다. 
- 유용한 디폴트 메서드를 제공할 수 있다. 

함수형 인터페이스를 작성한다면 `@FunctionalInterface` 애너테이션을 붙이자. 이유는 다음과 같다. 

1. 해당 클래스의 코드나 설명 문서를 읽을 이에게 그 인터페이스가 람다용으로 설계된 것임을 알려준다 
2. 해당 인터페이스가 추상 메서드를 오직 하나만 가지고 있어야 컴파일되게 해준다. 
3. 그 결과 유지 보수 과정에서 누군가 실수로 메서드를 추가하지 못하게 막아준다. 

>직접 만든 함수형 인터페이스에는 항상 `@FunctionalInterface` 애너테이션을 사용하라

서로 다른 함수형 인터페이스를 같은 위치의 인수로 받는 메서드들을 다중 정의해서는 안된다. 

클라이언트에게 불필요한 모호함을 안겨줘 문제가 생긴다. 

```java
public interface ExecutorService extends Executor {

	<T> Future<T> submit(Callable<T> task);

    <T> Future<T> submit(Runnable task, T result);

    Future<?> submit(Runnable task);
}
```
위와 같이 submit 메서드가 `Callable<T>`를 받는 것, `Runnable task`를 받는 것으로 다중정의해 올바른 메서드를 알려주기 위해 형변환해야할 때가 많이 생겼다. 

#### 출처

이펙티브 자바 3/E


[이펙티브 자바 github](https://github.com/WegraLee/effective-java-3e-source-code/tree/master)
