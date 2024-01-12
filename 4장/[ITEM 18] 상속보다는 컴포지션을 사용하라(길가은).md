# 상속

## 안전한 경우
- 상위 클래스, 하위클래스를 모두 같은 프로그래머가 통제하는 패키지 내
- 확장할 목적으로 설계되었고, 문서화도 잘 된 클래스

## 단점

### 메서드 호출과 달리 상속은 캡슐화를 깨뜨린다. 
상위 클래스가 어떻게 구현되느냐에 따라 하위 클래스의 동작에 이상이 생길 수 있다. 

```java
public class InstrumentedHashSet<E> extends HashSet<E> {
    // 추가된 원소의 수
    private int addCount = 0;

    public InstrumentedHashSet() {
    }

    public InstrumentedHashSet(int initCap, float loadFactor) {
        super(initCap, loadFactor);
    }

    @Override public boolean add(E e) {
        addCount++;
        return super.add(e);
    }

    @Override public boolean addAll(Collection<? extends E> c) {
        addCount += c.size();
        return super.addAll(c);
    }

    public int getAddCount() {
        return addCount;
    }

    public static void main(String[] args) {
        InstrumentedHashSet<String> s = new InstrumentedHashSet<>();
        s.addAll(List.of("틱", "탁탁", "펑"));
        System.out.println(s.getAddCount());
    }
}
```
위의 코드에서 `s.addAll(List.of("틱", "탁탁", "펑"));`을 하면 우리는 원소가 3개이므로 `addCount += c.size();`에서 `addCount`에 3이 더해져 `addCount`가 3이 되고 결론적으로 3이 출력될 것이라고 생각한다. 하지만, HashSet의 addAll의 내부를 살펴보면 아래와 같다. 

```java
    public boolean addAll(Collection<? extends E> c) {
        boolean modified = false;
        for (E e : c)
            if (add(e))
                modified = true;
        return modified;
    }
```
add를 반복해서 사용함으로써 addAll이 작동하고 있는 것이다. 그런데 우리는 `InstrumentedHashSet`을 구현하면서 `add` 메서드를 오버라이딩했다. 
```java
    @Override public boolean add(E e) {
        addCount++;
        return super.add(e);
    }
```
살펴보면 add가 될 때 마다 `addCount++;`이 실행되고 있는 것을 알 수 있다. 따라서 총 addCount는 6이 출력된다. 

이는 add를 재정의하지 않거나, addAll에서 원소를 순회하며 add를 호출하면 된다. 

```java
    @Override public boolean addAll(Collection<? extends E> c) {
        boolean modified = false;
        for (E e : c)
            if (add(e))
                modified = true;
        return modified;
    }
```

하지만, 시간도 더 들고 상위 클래스의 메서드 동작을 다시 구현해야해 어렵다. 또한 자칫 오류를 내거나 성능을 떨어뜨릴 수 있고, 하위 클래스에서는 접근할 수 없는 private 필드를 써야하는 상황이라면 이 방식으로는 구현이 불가능하다. 

### 요구사항이 변경될 때
상위 클래스에 새로운 메서드를 추가한다면? 보안 때문에 컬렉션에 추가된 모든 원소가 특정 조건을 만족해야만 하는 프로그램을 생각해보자. 
```java

    @Override public boolean add(E e) {
        if(Objects.requireNonNull(e))
        addCount++;
        return super.add(e);
    }

```
그렇다면 지금의 코드에선 원소를 추가할 때 무조건 `add`메서드를 사용하므로 이 메서드단에 검사를 하면 코드를 작성하면 된다. 하지만 상위 클래스에 `add`외의 다른 원소 추가 메서드가 생기게 된다면 이 검증을 거치치 않고 원소를 추가할 수 있게 될 것이다. 

### 오버라이딩을 하지말아보자.
위의 두 문제 모두 메서드 재정의로 인한 문제였다. 그렇다면, 메서드를 재정의하지 말고 새로운 메서드를 추가하는건 어떨까? 

상위클래스에 새 메서드가 추가 되었는데 내가 하위클래스에 추가한 메서드와 시그니처가 같고 반환타입이 다르다면 컴파일 조차 안되며 반환타입이 같다면 오버라이딩 한 것이므로 문제 해결이 안된다. 

## 컴포지션
새 클래스의 인스턴스 메서드들은 기존 클래스의 대응하는 메서드를 호출해 결과를 반환한다. 이를 전달이라하며, 새 클래ㅐ스의 메서드들을 전달 메서드라 한다. 

이를 통해 새 클래스는 기존 클래스의 내부 구현 방식의 영향에서 벗어나며 새로운 메서드가 추가되어도 전혀 영향을 받지 않는다. 

### 래퍼클래스
```java
public class InstrumentedSet<E> extends ForwardingSet<E> {
    private int addCount = 0;

    public InstrumentedSet(Set<E> s) {
        super(s);
    }

    @Override public boolean add(E e) {
        addCount++;
        return super.add(e);
    }
    @Override public boolean addAll(Collection<? extends E> c) {
        addCount += c.size();
        return super.addAll(c);
    }
    public int getAddCount() {
        return addCount;
    }

    public static void main(String[] args) {
        InstrumentedSet<String> s = new InstrumentedSet<>(new HashSet<>());
        s.addAll(List.of("틱", "탁탁", "펑"));
        System.out.println(s.getAddCount());
    }
}
```
### 재사용할 수 있는 전달 클래스

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
상속에서는 구체 클래스 각각을 따로 확장해야 하고, 지원하고 싶은 상위 클래스의 생성자 각각에 대응하는 생성자를 별도로 정의해줘야한다. 하지만 컴포지션은 한 번만 구현해두면 어떠한 Set 구현체라도 계츨할 수 있으며, 기존 생성자들과도 함께 사용할 수 있다. 

#### 단점

- 콜백 프레임워크와는 어울리지 않는다는 점이다. 

## 상속이 사용될 때

- 클래스 B가 클래스 A와 is-a 관계일 때만 클래스 A를 상속
   - B가 A인가?를 생각해보자. 


## 상속의 문제점

컴포지션을 써야할 상황에서 상속을 사용하는 것은 내부 구현을 불필요하게 노출하는 것이다. => API가 내부 구현에 묶이고 클래스의 성능도 영원히 제한된다. 또한, 클라이언트가 노출된 내부에 직접 접근할 수 있다. 

![](https://velog.velcdn.com/images/rlfrkdms1/post/2dc92b4f-4724-4f60-b14e-d57427b360e2/image.png)
Properties는 Hashtable을 상속받고 있는 것을 볼 수 있는데 Properties는 아래의 두가지 메서드를 모두 지원한다. 

```java
    public String getProperty(String key) {
        Object oval = map.get(key);
        String sval = (oval instanceof String) ? (String)oval : null;
        Properties defaults;
        return ((sval == null) && ((defaults = this.defaults) != null)) ? defaults.getProperty(key) : sval;
    }
    
    @Override
    public Object get(Object key) {
        return map.get(key);
    }
```
이는 같은 인스턴스에게 다른 결과를 가져올 수 있다. get은 Hashtable로 부터 재정의한 메서드이기 때문이다. 

가장 심각한 문제는 클라이언트에서 상위 클래스를 직접 수정하여 하위 클래스의 불변식을 해칠 수 있다는 점이다. Properties는 키와 값으로 문자열만 허용하도록 설계하려 했으나, Hashtable의 메서드들을 직접 호출하면 이를 깨버릴 수 있다. 

```java
    @Override
    public synchronized Object put(Object key, Object value) {
        return map.put(key, value);
    }
```
위와 같이 put 등의 메서드들이 그대로 살아있다. 

## 상속을 사용하기 전 생각해보자. 

- 확장하려는 클래스의 API에 아무런 결함이 없는가?
- 결함이 있다면, 이 결함이 내 클래스의 API까지 전파돼도 괜찮은가?

#### 상속

이펙티브 자바 3/E
