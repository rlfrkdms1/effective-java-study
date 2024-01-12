## 상속을 고려한 설계와 문서화

상속용 클래스는 재정의할 수 있는 메서드들을 내부적으로 어떻게 이용하는지(자기사용) 문서로 남겨야한다. 

메서드 주석에 `@implSpec`을 붙여주면 자바독 도구가 "Implementation Requirements"로 시작하는 절을 생성해준다. 이는 메서드의 내부동작 방식을 설명하는 곳이다. 
```java
   /**
     * {@inheritDoc}
     *
     * @implSpec
     * This implementation iterates over the elements in the collection,
     * checking each element in turn for equality with the specified element.
     *
     * @throws ClassCastException   {@inheritDoc}
     * @throws NullPointerException {@inheritDoc}
     */
    public boolean contains(Object o) {
        Iterator<E> it = iterator();
        if (o==null) {
            while (it.hasNext())
                if (it.next()==null)
                    return true;
        } else {
            while (it.hasNext())
                if (o.equals(it.next()))
                    return true;
        }
        return false;
    }
```

이처럼 내부 메커니즘을 문서로 남기는 것만이 상속을 위한 설계의 전부는 아니다. 
클래스의 내부 동작 과정 중간에 끼어들 수 있는 훅을 잘 선별하여 protected 메서드 형태로 공개해야 할 수도 있다. 

```java
    /**
     * Removes from this list all of the elements whose index is between
     * {@code fromIndex}, inclusive, and {@code toIndex}, exclusive.
     * Shifts any succeeding elements to the left (reduces their index).
     * This call shortens the list by {@code (toIndex - fromIndex)} elements.
     * (If {@code toIndex==fromIndex}, this operation has no effect.)
     *
     * <p>This method is called by the {@code clear} operation on this list
     * and its subLists.  Overriding this method to take advantage of
     * the internals of the list implementation can <i>substantially</i>
     * improve the performance of the {@code clear} operation on this list
     * and its subLists.
     *
     * @implSpec
     * This implementation gets a list iterator positioned before
     * {@code fromIndex}, and repeatedly calls {@code ListIterator.next}
     * followed by {@code ListIterator.remove} until the entire range has
     * been removed.  <b>Note: if {@code ListIterator.remove} requires linear
     * time, this implementation requires quadratic time.</b>
     *
     * @param fromIndex index of first element to be removed
     * @param toIndex index after last element to be removed
     */
    protected void removeRange(int fromIndex, int toIndex) {
        ListIterator<E> it = listIterator(fromIndex);
        for (int i=0, n=toIndex-fromIndex; i<n; i++) {
            it.next();
            it.remove();
        }
    }
    
   /**
     * Removes all of the elements from this list (optional operation).
     * The list will be empty after this call returns.
     *
     * @implSpec
     * This implementation calls {@code removeRange(0, size())}.
     *
     * <p>Note that this implementation throws an
     * {@code UnsupportedOperationException} unless {@code remove(int
     * index)} or {@code removeRange(int fromIndex, int toIndex)} is
     * overridden.
     *
     * @throws UnsupportedOperationException if the {@code clear} operation
     *         is not supported by this list
     */
    public void clear() {
        removeRange(0, size());
    }
```
List를 사용하는 사용자는 `removeRange`에 관심이 없지만, `removeRange`를 protected로 공개한 이유는 `clear`를 고성능으로 만들기 쉽게 하기 위해서이다. 

그렇다면 protected로 노출할 메서드는 어떻게 결정할까? 심사숙고해서 잘 예측해본 다음 실제 하위 클래스를 만들어 시험해보는 것이 최선이다. 

- 가능한 한 적어야한다. 
- 너무 적게 노출해 상속의 장점까지 없애지 않도록 주의한다. 

> 상속용 클래스를 시험하는 방법은 직접 하위 클래스를 만들어보는것이 **유일**하다.

### 상속용으로 설계한 클래스는 배포 전에 반드시 하위 클래스를 만들어 검증해야한다. 

상속시 꼭 필요한 protected 멤버를 놓쳤다면 하위 클래스를 작성할 때 그 빈자리가 확연히 드러난다. 거꾸로, 하위 클래스를 여러개 만듦에도 해당 멤버가 쓰이지 않는다면 private이었어야 할 가능성이 크다. 

하위 클래스를 3개정도 검증해봐야하며, 이 중 하나 이상은 제3자가 작성해봐야한다. 

### 상속용 클래스의 생성자는 직접적으로든 간접적으로든 재정의 가능 메서드를 호출해서는 안된다. 

```java
public class Super {
    // 잘못된 예 - 생성자가 재정의 가능 메서드를 호출한다.
    public Super() {
        overrideMe();
    }

    public void overrideMe() {
    }
}

public final class Sub extends Super {
    // 초기화되지 않은 final 필드. 생성자에서 초기화한다.
    private final Instant instant;

    Sub() {
        instant = Instant.now();
    }

    // 재정의 가능 메서드. 상위 클래스의 생성자가 호출한다.
    @Override public void overrideMe() {
        System.out.println(instant);
    }

    public static void main(String[] args) {
        Sub sub = new Sub();
        sub.overrideMe();
    }
}
```

1. `Sub sub = new Sub();`
2. `Super` 생성자 호출
3. `Super` 생성자 내의 `overrideMe` 호출
4. Sub에서 `overrideMe`를 오버라이딩 했으므로 Sub의 `overrideMe` 호출
5. `overrideMe`에서 instance를 출력 -> 아직 초기화 되지 않았으므로(Sub의 생성자가 호출되지 않음) null 출력
6. main의 `sub.overrideMe();` 호출
7. instance 출력

위의 `overrideMe`에서 만약 `System.out.println();`을 사용하지 않고 다른 메서드를 사용했다면 NPE가 터졌을 수도 있는 것이다. 

하지만 private, final, static 메서드는 재정의가 불가능하므로 생성자에서 안심하고 호출해도 된다. 

## Cloneable, Serializable

이들의 구현 클래스를 상속할 수 있게 설계하는 것은 좋지 않은 생각이다. 

>clone과 readObject 모두 직접적으로든 간접적으로든 재정의 가능 메서드를 호출해서는 안된다. 

- readObject : 하위 클래스의 상태가 역직렬화되기 전에 재정의한 메서드부터 호출하게 됨
- clone: 하위 클래스의 clone 메서드가 복제본의 상태를 수정하기 전에 재정의한 메서드 호출
   - 원본 객체에도 피해줄 수 있음
   
Serializable을 구현한 상속용 클래스가 readResolve나 writeReplace 메서드를 갖는다면 이 메서드들은 private이 아닌 protected로 선언해야한다. private으로 선언하면 하위 클래스에서 무시되기 때문이다. 

> 클래스를 상속용으로 설계하려면 엄청난 노력이 들고 그 클래스에 안기는 제약도 상당함을 알았다. 

## 어떻게 할까? 

### 상속용으로 설계하지 않은 클래스는 상속을 금지

상속을 금지하는 방법은 ITEM 17에서 다뤘다. 

클래스의 동작을 유지하면서 재정의 가능 메서드를 사용하는 코드를 제거할 수 있는 방법

기존 코드는 아래와 같다. 

```java
public class A {

    public A() {
        test();
    }

    public void test() {
        System.out.println("I'm in test method !!");
    }
    
}

public class B extends A {

    @Override
    public void test() {
        System.out.println("hi");
    }
    
}

```

```java
public class Main {
    public static void main(String[] args) {
        new B().test();
    }
}
```
위를 실행하면 아래와 같이 출력된다. 


```
hi
hi
```

그렇다면 도우미 메서드를 사용해보자. 
```java
public class A {

    public A() {
        도우미();
    }

    public void test() {
        도우미();
    }

    public void 도우미() {
        System.out.println("I'm in test method !!");
    }

}
```
기존에 재정의 대상이 되는 메서드의 본문을 도우미 메서드로 옮기고 해당 메서드가 사용되던 곳에 도우미 메서드를 호출해주었다. 다시 main 메서드를 실행해 결과를 보면 아래와 같다. 

```
I'm in test method !!
hi
```

#### 출처

이펙티브 자바 3/E
