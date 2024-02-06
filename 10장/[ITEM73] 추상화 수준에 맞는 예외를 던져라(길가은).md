메서드가 저수준 예외를 처리하지 않고 바깥으로 전파해버리면 수행하려는 일과 관련 없어보이는 예외가 튀어나온다. 이는 프로그래머를 당황시키고 내부 구현 방식을 드러내 윗 레벨 API를 오염시킨다. 

> 상위 계층에서는 저수준 예외를 잡아 자신의 추상화 수준에 맞는 예외로 바꿔 던져야한다. 

## 예외 번역

예외 번역이란 아래의 형식이다. 
```java
try {
	 ... //저수준 추상화 이용
} catch (LowerLevelException e) {
	//추상화 수준에 맞게 번역
    throw new HigherLevelException(...);
}
```
그럼 실제로 예외 번역이 쓰이는 사례를 살펴보자. 아래는 AbstarctSequentialList의 일부이다. 

```java
    /**
     * Returns the element at the specified position in this list.
     *
     * <p>This implementation first gets a list iterator pointing to the
     * indexed element (with {@code listIterator(index)}).  Then, it gets
     * the element using {@code ListIterator.next} and returns it.
     *
     * @throws IndexOutOfBoundsException {@inheritDoc}
     */
    public E get(int index) {
        try {
            return listIterator(index).next();
        } catch (NoSuchElementException exc) {
            throw new IndexOutOfBoundsException("Index: "+index);
        }
    }
```

`NoSuchElementException`를 `IndexOutOfBoundsException`로 번역하고 있는 것을 볼 수 있다. 

## 예외 연쇄

예외 번역시 저수준 예외가 디버깅에 도움이 되면 예외 연쇄를 사용하는게 좋다. 예외 연쇄란 문제의 근본 원인인 저수준 예외를 고수준 예외에 실어보내는 방식이다. 아래의 형식을 따른다. 

```java
try {
	 ... //저수준 추상화 이용
} catch (LowerLevelException cause) {
	//추상화 수준에 맞게 번역
    throw new HigherLevelException(cause);
}
```
고수준 예외의 생성자는 상위 클래스의 생성자에 이 원인을 건네줘 최종적으로 Throwable 생성자까지 건네지게 한다. 

```java
class HigerLevelException extends Exception {
	HigerLevelException(Throwable cause) {
    	super(cause);
    }
}
```
대부분의 표준 예외는 예외 연쇄용 생성자를 갖추고 있다. 그렇지 않더라도 Throwable의 initCause메서드를 이용해 원인을 직접 못박을 수 있다. 

예외 연쇄는 문제의 원인을 프로그램에서 접근할 수 있게 해주며, 원인과 고수준 예외의 스택 추적 정보를 잘 통합해준다. 

> 무턱대로 예외를 전파하는 것보다야 예외 번역이 우수한 방법이지만, 그렇다고 남용해서는 곤란하다. 

아래 계층에서의 예외를 피할 수 없다면, 상위 계층에서 그 예외를 조용히 처리하여 문제를 API 호출자에까지 전파하지 않는 방법이 있다. 이때 적절히 로그를 기록해두자. 


#### 출처

이펙티브 자바 3/E
