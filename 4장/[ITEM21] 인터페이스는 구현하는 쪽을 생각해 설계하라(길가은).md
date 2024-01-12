디폴트 메서드를 선언하면, 그 인터페이스를 구현한 후 디폴트 메서드를 재정의하지 않은 모든 클래스에서 디폴트 구현이 쓰이게 된다. 따라서 디폴트 메서드는 무작정 삽입된다. 

자바 8에서부터 사용된 것이다. 따라서 자가 7까지는 인터페이스에 메서드가 추가될 일이 없다고 생각하고 설계했다. 하지만 8이 되면서 주로 람다를 위해 핵심 컬렉션 인터페이스들에 다수의 디폴트 메서드가 추가되었다. 

문제는? 생각할 수 있는 모든 상황에서 불변식을 해치지 않는 디폴트 메서드를 작성하기란 어려운 법이다. 

```java
    default boolean removeIf(Predicate<? super E> filter) {
        Objects.requireNonNull(filter);
        boolean removed = false;
        final Iterator<E> each = iterator();
        while (each.hasNext()) {
            if (filter.test(each.next())) {
                each.remove();
                removed = true;
            }
        }
        return removed;
    }
```
이는 multi thread 환경에서 문제가 생기는데, 대표적인 예가 SynchronizedCollection에서이다. 여기선 모든 메서드에서 주어진 락 객체로 동기화한 후 내부 컬렉션 객체에 기능을 위임하는 래퍼클래스이다. 

이펙티브 자바가 쓰일 당시엔 removeIf를 재정의하고 있지 않아 default 메서드를 물려받게 되어 동기화를 해주지 못하고 있었다. 따라서 multi thread 환경에서는 문제가 생길 수 있다. 

그렇다면 어떻게 해야할까? 재정의하면된다. 

```java
@Override
        public boolean removeIf(Predicate<? super E> filter) {
            synchronized (mutex) {return c.removeIf(filter);}
        }
```
디폴트 메서드는 컴파일에 성공하더라도 기존 구현체에 런타임 오류를 일으킬 수 있다. 기존의 많은 코드에 디폴트 메서드가 추가 되어 영향을 받았기 때문이다. 

디폴트 메서드가 편리한 도구더라도 인터페이스를 설계할 때는 여전히 세심한 주의를 기울여야한다. 디폴트 메서드를 기존의 인터페이스에 추가하는 것은 굉장히 위험할 수 있는 일이기 때문이다. 

새로운 인터페이스라면, 릴리스 전에 반드시 테스트를 거쳐야한다. 인터페이스를 릴리스한 후라도 결함을 수정하는게 가능한 경우도 있겠지만, 절대 그 가능성에 기대서는 안된다. 

#### 출처 
이펙티브 자바 3/E
