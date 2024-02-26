한 메서드를 여러 스레드가 동시에 호출할 때 그 메서드가 어떻게 동작하느냐는 해당 클래스와 이를 사용하는 클라이언트 사이의 중요한 계약과 같으므로 API 문서에 작성해야한다. 

메서드 선언에 synchronized 한정자를 선언할지는 구현 이슈일 뿐 API에 속하지 않으므로 이 메서드가 synchronized 한정자를 가진다고 해서 스레드 안전하다고 믿기 어렵다. 

멀티스레드 환경에서도 API를 안전하게 사용하게 하려면 클래스가 지원하는 스레드 안전성 수준을 정확히 명시해야한다. 

- 불변 (immutable)
  - 이 클래스의 인스턴스는 마치 상수와 같아서 외부 동기화가 필요없다.
  - 예) String, Long, BigInteger 
- 무조건적 스레드 안전 (unconditionally thread-safe)
  - 이 클래스의 인스턴스는 수정될 수 있으나, 내부에서 충실히 동기화하여 별도의 외부 동기화가 필요없다.
  - synchronized 메서드가 아닌 비공개 락 객체를 사용한다.
  - 예) AtomicLong, ConcurrentHashMap
- 조건부 스레드 안전 (conditionally thread-safe)
  - 무조건적 스레드 안전과 같으나 일부 메서드는 동시에 사용하려면 외부 동기화가 필요하다
  - 주의해서 문서화해야 한다.
  - 어떤 순서로 호출할 때 외부 동기화가 필요한지, 그 순서로 호출하려면 어떤 락을 얻어야 하는지 알려줘야한다.
  - 클래스의 스레드 안정성은 보통 클래스의 문서화 주석에 기재하지만, 독특한   - 메서드라면 해당 메서드에 문서화하자.
  - 예) Collections.synchronizedMap, Collections.synchronizedSet 등등이 반환한 컬렉션들
  ```java
      /**
     * Returns a synchronized (thread-safe) map backed by the specified
     * map.  In order to guarantee serial access, it is critical that
     * <strong>all</strong> access to the backing map is accomplished
     * through the returned map.<p>
     *
     * It is imperative that the user manually synchronize on the returned
     * map when traversing any of its collection views via {@link Iterator},
     * {@link Spliterator} or {@link Stream}:
     * <pre>
     *  Map m = Collections.synchronizedMap(new HashMap());
     *      ...
     *  Set s = m.keySet();  // Needn't be in synchronized block
     *      ...
     *  synchronized (m) {  // Synchronizing on m, not s!
     *      Iterator i = s.iterator(); // Must be in synchronized block
     *      while (i.hasNext())
     *          foo(i.next());
     *  }
     * </pre>
     * Failure to follow this advice may result in non-deterministic behavior.
     *
     * <p>The returned map will be serializable if the specified map is
     * serializable.
     *
     * @param <K> the class of the map keys
     * @param <V> the class of the map values
     * @param  m the map to be "wrapped" in a synchronized map.
     * @return a synchronized view of the specified map.
     */
    public static <K,V> Map<K,V> synchronizedMap(Map<K,V> m) {
        return new SynchronizedMap<>(m);
    }
    ```
    위는 synchronizedMap의 일부이다. 주석 중 다음의 부분에 주목해보자. 
    ```
         * It is imperative that the user manually synchronize on the returned
     * map when traversing any of its collection views via {@link Iterator},
     * {@link Spliterator} or {@link Stream}:
     * <pre>
     *  Map m = Collections.synchronizedMap(new HashMap());
     *      ...
     *  Set s = m.keySet();  // Needn't be in synchronized block
     *      ...
     *  synchronized (m) {  // Synchronizing on m, not s!
     *      Iterator i = s.iterator(); // Must be in synchronized block
     *      while (i.hasNext())
     *          foo(i.next());
     *  }
     * </pre>
     * Failure to follow this advice may result in non-deterministic behavior.
     ```
     마지막 줄에서 위를 따르지 않으면 행동을 예측할 수 없는 결과를 얻을 것이라고 적혀있다. 
- 스레드 안전하지 않음 (not thread-safe)
  - 클래스의 인스턴스가 수정될 수 있다
  - 동시에 사용하려면 클라이언트가 별도의 동기화를 수행해야 한다.
  - 예) ArrayList, HashMap 등의 기본 컬렉션
- 스레드 적대적 (thread-hostile)
  - 이 클래스는 모든 메서드 호출을 외부 동기화로 감싸더라도 멀티스레드 환경에서 안전하지 않다.
  - 고의로 만드는 사람은 없지만 동시성을 고려하지 않고 작성하다보면 우연히 만들어질 수 있다.
  - 예) generateSerialNumber에서 내부 동기화를 생략했을 때



클래스가 외부에서 사용할 수 있는 락을 제공하면 메서드 호출을 원자적으로 수행할 수 있으나, 이 유연성엔 대가가 따른다. 내부에서 처리하는 고성능 동시성 제어 메커니즘과 혼용할 수 없게 되어 동시성 컬렉션과는 함께 사용하지 못한다. 또한 클라이언트가 공개된 락을 오래 쥐고 놓지 않는 서비스 거부 공격(DOS)을 수행할 수도 있다. 

이 공격을 막으려면 synchronized 메서드 대신 비공개 락 객체를 사용해야한다. 

```java
private final Object lock = new Object();

public void foo() {
	synchronized(lock) {
		...
	}
}
```
비공개 락 객체를 사용하면 클라이언트가 동기화에 관여할 수 없다. 또한 락 필드는 항상 final로 선언하자. 또한, 무조건적 스레드 안전 클래스에서만 사용할 수 있다. 

#### 출처

이펙티브 자바 3/E
