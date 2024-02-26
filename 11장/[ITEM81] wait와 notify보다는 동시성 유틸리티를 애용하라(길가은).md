# 동시성 유틸리티
> wait와 notify는 올바르게 사용하기가 아주 까다로우니 고수준 동시성 유틸리티를 사용하자. 

- java.util.concurrent의 고수준 유틸리티
  - 실행자 프레임워크
  - 동시성 컬렉션
  - 동기화 장치
  
## 동시성 컬렉션

> 동시성 컬렉션에서 동시성을 무력화하는 건 불가능하며, 외부에서 락을 추가로 사용하면 오히려 속도가 느려진다. 

여러 기본 동작을 하나의 원자적 동작으로 묶는 상태 의존적 수정 메서드들을 지원한다. 
`Map`의 `putIfAbsent(key, value)`는 주어진 키에 매핑된 값이 아직 없을 때만 새값을 집어넣고 기존 값이 있었다면 그 값을 반환하고 없었다면 unll을 반환한다. 이를 이용해 안전한 정규화 맵을 만들어보자.

### ConcurrentMap

```java
public class Intern {
 
    private static final ConcurrentMap<String, String> map =
            new ConcurrentHashMap<>();

    public static String intern(String s) {
        String previousValue = map.putIfAbsent(s, s);
        return previousValue == null ? s : previousValue;
    }
````
위의 코드는 String.intern의 동작을 흉내낸 것이다. map은 get 같은 검색 기능에 최적화되어있으므로 get을 먼저 호출해 putIfAbsent를 필요할 때만 호출해보자. 

```java
    public static String intern(String s) {
        String result = map.get(s);
        if (result == null) {
            result = map.putIfAbsent(s, s);
            if (result == null)
                result = s;
        }
        return result;
    }
```
ConcurrnetMap은 동시성이 뛰어나며 속도도 무척빠르다. 

따라서 이제는 동기화한 컬렉션 대신 동시성 컬렉션을 사용하자. 예로 Collections.synchronizedMap보다는 ConcurrentHashMap을 사용하는게 낫다. 

### BlockingQueue
컬렉션 인터페이스 중 일부는 작업이 성공적으로 완료될 때 까지 기다리도록(차단) 확장되었다. 

```java
public interface BlockingQueue<E> extends Queue<E> {

    /**
     * Retrieves and removes the head of this queue, waiting if necessary
     * until an element becomes available.
     *
     * @return the head of this queue
     * @throws InterruptedException if interrupted while waiting
     */
    E take() throws InterruptedException;
}
```
BlockingQueue의 take는 큐의 첫 원소를 꺼내는데 큐가 비었다면 새로운 원소가 추가될 때까지 기다린다. 

## 동기화 장치
동기화 장치는 스레드가 다른 스레드를 기다릴 수 있게하여, 서로 작업을 조율할 수 있게 해준다. 

### CountDownLatch
일회성 장벽으로 하나 이상의 스레드가 또 다른 하나 이상의 스레드 작업이 끝날 때까지 기다리게 한다. 
생성자에서 받는 int 값이 래치의 coundDown 메서드를 몇번 호출해야 대기 중인 스레드들을 깨우는지를 결정한다. 

이 장치를 사용해서 어떤 동작이 수행되는 시간을 재는 간단한 프레임워크를 구축해보자. 

```java
public class ConcurrentTimer {
    private ConcurrentTimer() { } // 인스턴스 생성 불가

    public static long time(Executor executor, int concurrency,
                            Runnable action) throws InterruptedException {
        CountDownLatch ready = new CountDownLatch(concurrency);
        CountDownLatch start = new CountDownLatch(1);
        CountDownLatch done  = new CountDownLatch(concurrency);

        for (int i = 0; i < concurrency; i++) {
            executor.execute(() -> {
                ready.countDown(); // 타이머에게 준비를 마쳤음을 알린다.
                try {
                    start.await(); // 모든 작업자 스레드가 준비될 때까지 기다린다.
                    action.run();
                } catch (InterruptedException e) {
                    Thread.currentThread().interrupt();
                } finally {
                    done.countDown();  // 타이머에게 작업을 마쳤음을 알린다.
                }
            });
        }

        ready.await();     // 모든 작업자가 준비될 때까지 기다린다.
        long startNanos = System.nanoTime();
        start.countDown(); // 작업자들을 깨운다.
        done.await();      // 모든 작업자가 일을 끝마치기를 기다린다.
        return System.nanoTime() - startNanos;
    }
}
```
time 메서드는 동시성 수준(동작을 몇개나 동시에 수행할 수 있는지를 뜻함)을 매개변수로 받는다. 

시간 간격을 잴 때는 항상 더 정확하고 정밀하며 시스템의 실시간 시계의 시간 보정에 영향을 받지 않는 `System.nanoTime`을 사용하자. 

위의 예제에선 CountDownLatch를 3개 사용했지만 이는 CyclicBarrier인스턴스 하나로 대체할 수 있다. 하지만 이해하기는 더 어려울 것이다. 

## 어쩔 수 없이 wait와 notify를 써야할 땐?
wait : 스레드가 어떤 조건이 충족되기를 기다리게 할 때 사용, 락 객체의 wait는 반드시 그 객체를 잠근 동기화 영역 내에서 호출

```java
synchronized (obj) {
    while (<조건이 충족되지 않았다>) {
        obj.wait(); // 락을 놓고, 깨어나면 다시 잡는다.
    }

    ... // 조건이 충족됐을 때의 동작을 수행한다.
}
```
wait 메서드를 사용할 때는 반드시 대기 반복문 관용구를 사용하고 반복문 밖에서는 절대로 호출하지 말자. 이 반복문은 wait 호출 전후로 조건이 만족하는지를 검사하는 역할을 한다. 이는 응답 불가 상태를 예방하기 위해서이다.

한편, 대기한 이후에 조건을 검사하여 조건을 충족하지 않았을 때 안전 실패 예방을 위해 다시 대기하게 하는 경우도 있는데, 이는 조건이 만족되지 않아도 스레드가 깨어날 수 있는 상황이 몇 가지 있기 때문이다.

- 스레드가 notify를 호출한 다음 대기 중이던 스레드가 깨어나는 사이에 다른 스레드가 락을 얻어 그 락이 보호하는 상태를 변경
- 조건이 만족되지 않았음에도 다른 스레드가 실수 혹은 악의적으로 notify 호출. 
   - 공개된 객체를 락으로 사용해 대기하는 클래스는 이러한 위험에 노출됨. 
   - 외부에 노출된 객체의 동기화된 메서드 안에서 호출하는 wait는 모두 이문제에 영향을 받음
- 깨우는 스레드는 지나치게 관대해, 대기 중인 스레드 중 일부만 조건이 충족되어도 notifyAll을 호출해 모든 스레드를 깨울 수 있음
- 대기중인 스레드가 notify없이도 깨어나는 경우가 있음(허위 각성)

일반적으로 언제나 notifyAll을 사용하자. 

#### 출처

이펙티브 자바 3/E
