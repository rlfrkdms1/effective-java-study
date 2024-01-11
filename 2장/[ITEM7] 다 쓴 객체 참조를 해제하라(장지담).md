가비지 컬렉터가 있는 언어에서는 메모리 관리를 신경쓰지 않아도 될 것 같지만 아니다. 
다 쓴 참조를 살려두는 경우 메모리 누수가 발생한다.
다 쓴 참조를 살려두면 GC는 다 쓴 참조의 객체뿐만 아니라 그 객체가 참조하는 모든 객체를 회수하지 못한다.

# 메모리 누수
```java
public class Stack {
    private static final int MAX_SIZE = 100;
    private Object[] stack;
    private int size;

    public Stack() {
        stack = new Object[MAX_SIZE];
        size = 0;
    }

    public Object pop() {
        if (size == 0) {
            throw new EmptyStackException();
        }
        return stack[--size];
    }
}
```
- `pop()`에서 stack 배열의 활성 영역 밖의 참조는 다 쓴 참조다.
- GC가 다 쓴 참조인 Object들을 회수하지 못한다.
```java
    public Object pop() {
        if (size == 0) {
            throw new EmptyStackException();
        }
        Object pop = stack[--size];
        // 다 쓴 참조 해제
        stack[size] = null;
        return pop;
    }
```
- 다 쓴 참조를 null 처리(참조 해제) 해줘야 한다.
- null 처리한 참조를 실수로 사용하려 할 경우 NPE가 발생한다.

# null 처리가 필요한 상황
- null 처리는 필요한 경우에만 해야 한다. 불필요한 null 처리는 코드를 지저분하게 만든다.

### 자기 메모리를 직접 관리하는 클래스
- 다 쓴 참조 여부를 프로그래머만 알 수 있다
- null 처리를 통해 GC에게 알려줘야 한다.
- 위의 Stack 클래스

### 캐시
- 객체 참조를 캐시에 넣고 방치하는 경우가 많다
- 외부에서 key를 참조하는 동안에만 entry가 살아있는 캐시가 필요한 경우에는 WeakHashMap을 사용하자
```java
        Map<Dog, String> hm = new WeakHashMap<>();
        Dog d1 = new Dog(1);
        Dog d2 = new Dog(2);
        hm.put(d1,"a");
        hm.put(d2,"b");

        d1 = null;

        System.gc();

        hm.entrySet().stream().forEach(s -> System.out.println(s));
```
![image](https://github.com/rlfrkdms1/effective-java-study/assets/96513365/fe62c3e0-94fe-4961-a791-3269020212ba)

![image](https://github.com/rlfrkdms1/effective-java-study/assets/96513365/8e43788d-759f-4733-8b2a-9e22d8cc3a70)
key인 Dog@466의 외부참조 d1이 `d1=null`로 변경되었다. 
![image](https://github.com/rlfrkdms1/effective-java-study/assets/96513365/3223c2bc-fca8-49f8-b084-7d4bcdfcf6e3)
외부에서 key를 참조하지 않기 때문에 GC에 의해 수거되었다.

- 보통 캐시 엔트리의 유효 기간을 정확히 정의하기 어렵기 때문에 시간이 지날수록 엔트리 가치를 떨어트리는 방법을 사용한다. LinkedHashMap은 새 엔트리를 추가할 때 마다 부수 작업으로 가장 오래된 엔트리를 제거하는 방식을 사용한다. 

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
        // 중략.. 대충 값 추가하는 코드 .. 
        afterNodeInsertion(evict);
        return null;
    }
```
HashMap의 일부로, put에서(새 엔트리를 추가할 때) putVal을 호출하고 putVal에서 afterNodeInsertion을 호출한다.
```java
    void afterNodeInsertion(boolean evict) { // possibly remove eldest
        LinkedHashMap.Entry<K,V> first;
        if (evict && (first = head) != null && removeEldestEntry(first)) {
            K key = first.key;
            removeNode(hash(key), key, null, false, true);
        }
    }
```
HashMap을 상속하는 LinkedHashMap의 afterNodeInsertion이다. removeEldestEntry의 반환값에 따라 가장 오래된 엔트리를 제거할 지 결정한다.
```java
    protected boolean removeEldestEntry(Map.Entry<K,V> eldest) {
        return false;
    }
```
다만 LinkedHashMap의 removeEldestEntry에서 항상 false를 반환한다.
```java
        LinkedHashMap<Integer, Integer> hm = new LinkedHashMap<>(1000, 0.75f, true) {

            private final static int MAX = 3;

            @Override
            protected boolean removeEldestEntry(java.util.Map.Entry<Integer, Integer> eldest) {
                return size() >= MAX;
            }
        };

        hm.put(1, 1);
        hm.put(2, 2);
        System.out.println(hm.containsValue(1)); // true
        hm.put(3, 3);
        System.out.println(hm.containsValue(1)); // false
```
이렇게 LinkedHashMap의 익명 클래스로 removeEldestEntry를 재정의해주면 엔트리를 추가할 때 마다 가장 오래된 엔트리를 제거할 지 판단해 제거해야한다면 제거하는 것을 확인할 수 있다. 
### 콜백 혹은 리스너 
> 콜백(혹은 리스너) : 특정 이벤트가 발생하면 콜백함수를 호출하는 것
- 클라이언트가 콜백을 등록만 하고 해지하지 않으면 콜백이 쌓인다.
- 콜백을 약한참조로 저장하면 가비지 컬렉터의 수거 대상이 된다.
> 약한참조 : WeakReference 클래스로 생성 가능, 참조하는 객체가 null이 되면 가비지 컬렉터의 수거 대상이 됨
```java
        Dog d = new Dog(1);
        WeakReference weak = new WeakReference<>(d);
        d = null;
        System.gc();
```
- 콜백을 WeakHashMap에 key로 저장하는 방법을 사용할 수 있다. (WeakHashMap 예시는 위에 있다.)


