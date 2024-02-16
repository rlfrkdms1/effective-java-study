# 과도한 동기화

- 성능을 떨어뜨림
- 교착상태에 빠뜨림
- 예측할 수 없는 동작을 낳음

> 응답 불가와 안전 실패를 피하려면 동기화 메서드나 동기화 블록 안에서는 제어를 절대로 클라이언트에 양도하면 안 된다. 

## 정확성

예시를 살펴보자 !

```java
public class ForwardingSet<E> implements Set<E> {
    private final Set<E> s;
    public ForwardingSet(Set<E> s) { this.s = s; }

    public void clear()               { s.clear();            }
    public boolean contains(Object o) { return s.contains(o); }
    public boolean isEmpty()          { return s.isEmpty();   }
    public int size()                 { return s.size();      }
    public Iterator<E> iterator()     { return s.iterator();  }
    public boolean add(E e)           { return s.add(e);      }
    public boolean remove(Object o)   { return s.remove(o);   }
    public boolean containsAll(Collection<?> c)
    { return s.containsAll(c); }
    public boolean addAll(Collection<? extends E> c)
    { return s.addAll(c);      }
    public boolean removeAll(Collection<?> c)
    { return s.removeAll(c);   }
    public boolean retainAll(Collection<?> c)
    { return s.retainAll(c);   }
    public Object[] toArray()          { return s.toArray();  }
    public <T> T[] toArray(T[] a)      { return s.toArray(a); }
    @Override public boolean equals(Object o)
    { return s.equals(o);  }
    @Override public int hashCode()    { return s.hashCode(); }
    @Override public String toString() { return s.toString(); }
}
```
Set의 기능들을 전달하는 전달 클래스인 ForwardingSet을 위와 같이 정의하자. 

```java
import java.util.*;
import java.util.concurrent.CopyOnWriteArrayList;

public class ObservableSet<E> extends ForwardingSet<E> {
    public ObservableSet(Set<E> set) { super(set); }

		private final List<SetObserver<E>> observers
            = new ArrayList<>();

    public void addObserver(SetObserver<E> observer) {
        synchronized(observers) {
            observers.add(observer);
        }
    }

    public boolean removeObserver(SetObserver<E> observer) {
        synchronized(observers) {
            return observers.remove(observer);
        }
    }

    private void notifyElementAdded(E element) {
        synchronized(observers) {
            for (SetObserver<E> observer : observers)
                observer.added(this, element);
        }
    }

    @Override 
		public boolean add(E element) {
        boolean added = super.add(element);
        if (added)
            notifyElementAdded(element);
        return added;
    }

    @Override 
		public boolean addAll(Collection<? extends E> c) {
        boolean result = false;
        for (E element : c)
            result |= add(element);  // notifyElementAdded를 호출한다.
        return result;
    }
}
```
Set의 래퍼 클래스로 집합에 원소가 추가되면 알림을 받는다. 이를 관찰자 패턴이라고 한다. `removeObserver`와 `addObserver`를 통해 구독을 신청하거나 해지할 수 있다. 두 경우 모두 아래의 콜백 인터페이스의 인스턴스를 메서드에 건넨다. 

```java
@FunctionalInterface
public interface SetObserver<E> {
    void added(ObservableSet<E> set, E element);
}
```
자 그럼 이제 사용해보자.

```java
public class Test1 {
    public static void main(String[] args) {
        ObservableSet<Integer> set =
                new ObservableSet<>(new HashSet<>());

        set.addObserver((s, e) -> System.out.println(e));

        for (int i = 0; i < 100; i++)
            set.add(i);
    }
}
```
set에 원소가 추가될 경우 출력하도록 하고 0부터 99까지의 수를 set에 넣어보았다. 결과는 아래와 같다. 

```
0
1
2
3
4
5
6
...
99
```

### ConcurrentModificationException
그렇다면 이제 원소가 추가되었을 때 출력하는 기능에 더해 값이 23이라면 자기 자신(구독해지)을 제거하는 관찰자를 추가해보자. 

```java
public class Test2 {
    public static void main(String[] args) {
        ObservableSet<Integer> set =
                new ObservableSet<>(new HashSet<>());

        set.addObserver(new SetObserver<>() {
            public void added(ObservableSet<Integer> s, Integer e) {
                System.out.println(e);
                if (e == 23) // 값이 23이면 자신을 구독해지한다.
                    s.removeObserver(this);
            }
        });

        for (int i = 0; i < 100; i++)
            set.add(i);
    }
}
```
위와 같이 작성할 수 있다. 하지만 이를 실행하면 23까지 실행하고 ConcurrentModificationException을 던진다. notifyElementAdded가 호출되었기 때문이다. notifyElementAdded를 보면 observer들을 가지고 있는 리스트인 observers를 순회하며 added메서드를 호출한다. 우리는 added 메서드 내에 observer가 삭제되도록 정의했으므로 observers 리스트가 순회되던 도중 삭제 연산이 수행되어 예외가 던져진 것이다. 

이 순회는 동기화 블록 안에 있으므로 동시 수정이 일어나지 않도록 보장하지만, 정작 자신이 콜백을 거쳐 되돌아와 수정하는 것 까지 막지 못한다. 

### 교착상태

그렇다면 removeObserver를 직접 호출하지말고 실행자 서비스를 이용해보자. 

```java
public class Test3 {
    public static void main(String[] args) {
        ObservableSet<Integer> set =
                new ObservableSet<>(new HashSet<>());

        set.addObserver(new SetObserver<>() {
            public void added(ObservableSet<Integer> s, Integer e) {
                System.out.println(e);
                if (e == 23) {
                    ExecutorService exec =
                            Executors.newSingleThreadExecutor();
                    try {
                        exec.submit(() -> s.removeObserver(this)).get();
                    } catch (ExecutionException | InterruptedException ex) {
                        throw new AssertionError(ex);
                    } finally {
                        exec.shutdown();
                    }
                }
            }
        });

        for (int i = 0; i < 100; i++)
            set.add(i);
    }
}
```
위 프로그램을 실행하면 스레드가 s.removeObserver를 호출하면 관찰자를 잠그려 시도 하지만, 메인스레드가 이미 락을 쥐고 있어 얻을 수 없다. 이와 동시에 메인 스레드는 백그라운드 스레드가 관찰자를 제거하길 기다린다. 따라서 교착상태에 빠진다. 

그렇다면 똑같은 상황이나, 불변식이 임의로 깨졌다면 어떻게 될까? 교착상태에 빠지진 않는다. 첫번째 예에서라면 락이 보호하는 데이터에 대해 개념적으로 관련이 없는 다른 작업이 진행 중이더라도 락 획득에 성공한다. 따라서 좋지 않는 결과가 빚어질 수 있다. 

## 해결법

### 동기화 블록 바깥으로 !
해결해보자. 리스트를 순회하는 코드를 동기화 블록 바깥으로 옮겨서 사용하면 된다. 
```java
private void notifyElementAdded(E element) {
        List<SetObserver<E>> snapshot = null;
        synchronized (observers) {
            snapshot = new ArrayList<>(observers);
        }
        for (SetObserver<E> observer : snapshot)
            observer.added(this, element);
    }
```
이제 안전하게 순회할 수 있다. 

### CopyOnWriteArrayList

이러한 방법보다 더 나은 방법도 있다. CopyOnWriteArrayList를 사용하면 된다. 이는 내부를 변경하는 작업은 항상 깨끗한 복사본을 만들어 수행하도록 구현되었다. 순회를 할 때는 락이 필요없으니 매우 빠르다. 아래와 같이 수정해보자. 

```java
private final List<SetObserver<E>> observers = new CopyOnWriteArrayList<>();

public void addObserver(SetObserver<E> observer) {
    observers.add(observer);
}

public boolean removeObserver(SetObserver<E> observer) {
    return observers.remove(observer);
}

private void notifyElementAdded(E element) {
	for (SetObserver<E> observer : observers)
		observer.added(this, element);
}
```

## 규칙

이런 일이 일어나지 않으려면 어떻게 해야할까?
동기화 영역에서는 가능한 한 일을 적게 하는 것이다.

## 성능

과도한 동기화가 초래하는 진짜 비용은 병렬로 실행할 기회를 잃고, 모든 코어가 메모리를 일관되게 보기 위한 지연시간이다. 가상머신의 코드 최적화를 제한한다는 점도 숨은 비용이다. 

## 어떻게 해야할까?

따라서 가변 클래스를 작성하려거든 다음 두 선택지 중 하나를 따르자. 
- 동기화를 전혀 하지말고, 외부에서 알아서 동기화 하게 하자
   - java.util
- 동기화를 내부에서 수행해 스레드 안전한 클래스로 만들자
   - 클라이언트가 외부에서 객체 전체에 락을 거는 것보다 동시성을 월등히 개선할 수 있을 때만
   - java.util.concurrent
   
### 클래스를 내부에서 동기화하기로 했다면?

다양한 기법을 동원할 수 있다. 
- 락 분할
- 락 스트라이핑
- 비차단 동시성 제어

여러 스레드가 호출할 가능성이 있는 메서드가 정적 필드를 수정한다면 그 필드를 사용하기 전에 반드시 동기해야한다. 


#### 출처

이펙티브 자바 3/E
