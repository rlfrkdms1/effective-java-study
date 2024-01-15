아래의 스택 코드는 제네릭 타입이어야 마땅하다. 

```java
public class Stack {
    private Object[] elements;
    private int size = 0;
    private static final int DEFAULT_INITIAL_CAPACITY = 16;

    public Stack() {
        elements = new Object[DEFAULT_INITIAL_CAPACITY];
    }

    public void push(Object e) {
        ensureCapacity();
        elements[size++] = e;
    }


    private void ensureCapacity() {
        if (elements.length == size)
            elements = Arrays.copyOf(elements, 2 * size + 1);
    }

    public Object pop() {
        if (size == 0)
            throw new EmptyStackException();
        Object result = elements[--size];
        elements[size] = null; // 다 쓴 참조 해제
        return result;
    }

    public static void main(String[] args) {
        Stack stack = new Stack();
        for (String arg : args)
            stack.push(arg);

        while (true)
            System.err.println(stack.pop());
    }
}
```
지금 상태에서는 스택에서 꺼낸 객체를 형변환할 때 런타임 오류가 날 위험이 있다. 그렇다면, 제네릭으로 바꿔보자. 타입 이름으로는 보통 E를 사용한다. 

```java
package item29;

import java.util.Arrays;
import java.util.EmptyStackException;

public class Stack<E> {
    private E[] elements;
    private int size = 0;
    private static final int DEFAULT_INITIAL_CAPACITY = 16;

    public Stack() {
        elements = new E[DEFAULT_INITIAL_CAPACITY];
    }

    public void push(E e) {
        ensureCapacity();
        elements[size++] = e;
    }


    private void ensureCapacity() {
        if (elements.length == size)
            elements = Arrays.copyOf(elements, 2 * size + 1);
    }

    public E pop() {
        if (size == 0)
            throw new EmptyStackException();
        E result = elements[--size];
        elements[size] = null; // 다 쓴 참조 해제
        return result;
    }

}
```
Object를 모두 E로 바꾼 셈이다. 문제점이 있다. 

```java
elements = new E[DEFAULT_INITIAL_CAPACITY];
```
에서 컴파일 에러가 뜨는 것이다. 실체화 불가 타입으로는 배열을 만들 수 없기 때문이다. 따라서 두가지 선택지가 있다. 

1. Object 배열을 생성한 다음 제네릭 배열로 형변환
   ```java
   elements = (E[]) new Object[DEFAULT_INITIAL_CAPACITY];
   ```
   오류는 없겠지만, 경고를 내보낼 것이고 타입 안전하지 않다. 
   따라서  타입 안전성을 확인해보자. 문제의 배열 elements는 private 필드에 저장되고, 클라이언트로 반환되거나 다른 메서드에 전달되는 일이 전혀 없다. push 메서드를 통해 배열에 저장되는 원소의 타입은 항상 E다. 따라서 이 비검사 형변환은 안전하다. 
   안전을 확인했으므로 `@SuppressWarning` 애너테이션으로 경고를 숨긴다. 
   ```java
    // 배열 elements는 push(E)로 넘어온 E 인스턴스만 담는다.
    // 따라서 타입 안전성을 보장하지만,
    // 이 배열의 런타임 타입은 E[]가 아닌 Object[]다!
    @SuppressWarnings("unchecked")
    public Stack() {
        elements = (E[]) new Object[DEFAULT_INITIAL_CAPACITY];
    }
    ```
2. elements 필드의 타입을 `E[]`에서 `Object[]`로 바꾸는 것이다. 생성자에서 `Object[]`로 선언하고, `pop()`에서 형변환을 한다. 
   ```java
   elements = new Object[DEFAULT_INITIAL_CAPACITY];
   ```
   또한 이를 통한 경고는 아래와 같이 숨길 수 있을 것이다. 
   ```java
       // 비검사 경고를 적절히 숨긴다.
    public E pop() {
        if (size == 0)
            throw new EmptyStackException();

        // push에서 E 타입만 허용하므로 이 형변환은 안전하다.
        @SuppressWarnings("unchecked") E result =
                (E) elements[--size];

        elements[size] = null; // 다 쓴 참조 해제
        return result;
    }
    ```

둘의 차이는 무엇일까? 

첫번째 방법
- 가독성이 더 좋다. 
  - 배열의 타입을 `E[]`로 선언해 오직 `E` 타입 인스턴스만 받음을 확실히 어필
  - 더 짧음
- 형변환 한번 - 배열 생성시
- 현업에서 선호
- 배열의 런타임 타입이 컴파일타임 타입과 달라 힙오염
   컴파일 타임엔 타입이 E이나, 런타임시엔 타입이 `Object[]`이다.
   ```java
       public Stack() {
        String simpleName = this.elements.getClass().getSimpleName();
        System.out.println("element's class = " + simpleName);
    }
    ```
    ```
    element's class = Object[]
    ```

두번째 방법
- 배열에서 원소를 읽을 때 마다 형변환



대다수의 제네릭 타입은 타입 매개변수에 아무런 제약을 두지 않는다. 따라서 어떤 참조 타입으로도 Stack을 만들 수 있지만, **기본 타입**은 사용할 수 없다.

O : `Stack<Object>, Stack<int[]>, Stack<List<String>>`
X : `Stack<int>, Stack<double>`

타입 매개변수에 제약을 두는 제네릭 타입
`class DelayQueu<E extends Delayed> implements BlockingQueue<E>`
타입 매개변수 목록인 `<E extends Delayed>`는 java.util.concurrent.Delayed의 하위 타이반 받는다는 뜻이다. 따라서 DelayQueue 자신과 DelayQueue를 사용하는 클라이언트는 DelayQueue의 원소에서 곧바로 Delayed 클래스의 메서드를 호출할 수 있으며 이러한 타입 매개변수 E를 한정적 타입 매개변수라 한다. 

모든 타입은 자기 자신의 하위 타입이므로 `DelayQueue<Delayed>`로도 사용할 수 있음을 기억해두자. 

#### 출처 

이펙티브 자바 3/E
