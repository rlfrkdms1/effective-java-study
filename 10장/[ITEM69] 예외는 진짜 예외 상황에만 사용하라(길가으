```java
public class Mountain {

    private String name;

    public Mountain(String name) {
        this.name = name;
    }

    public void climb() {
        System.out.println("등산 시작 !");
    }
}
``` 
위와 같이 "산" 클래스가 있을 때 여러산을 등산해보자.

```java
        Mountain[] mountains = {new Mountain("Hallasan"), new Mountain("Seoraksan"), new Mountain("Jirisan")};
        try {
            int i = 0;
            while (true) {
                mountains[i++].climb();
            }
        } catch (ArrayIndexOutOfBoundsException e) {

        }
```
위의 코드는 mountains라는 배열에 산들을 모아놓고 무한 반복문을 통해 `ArrayIndexOutOfBoundsException` 예외가 던져질 때 까지 배열을 순회하고 있다. 아찔하지 않은가.. for-each문, 그냥 for문 등 개선할 방향은 많다. 

```java
        for (Mountain mountain : mountains) {
            mountain.climb();
        }

        for (int i = 0; i < mountains.length; i++) {
            mountains[i].climb();
        }
```

예외를 통한 순회는 프로그래머의 의도도 파악하기 쉽지 않고, 예외를 던지기 때문에 try-catch문을 사용해야된다는 단점 또한 존재한다. 하지만 for-each문을 사용하면 가독성도 좋다. 

그렇다면 왜 예외를 던졌을까? 

JVM은 배열에 접근할 때마다 경계를 넘지 않는지 검사하는데, 일반적인 반복문도 배열의 경계에 도달하면 종료한다. 따라서 검사가 중복된다고 생각하여 생략한 것이다. 하지만 이는 잘못된 접근이다. 

1. 예외는 예외 상황에 쓸 용도로 설계되었으므로 JVM 구현자 입장에서는 명확한 검사만큼 빠르게 만들어야 할 동기가 약하다. 
2. 코드를 try-catch 블록 안에 넣으면 JVM이적용할 수 있는 최적화가 제한된다. 
3. 배열을 순회하는 표준 관용구는 앞서 걱정한 중복 검사를 수행하지 않는다. JVM이 알아서 최적화해 없애준다. 

예외로 배열을 순회할 때 문제점은 또 있다. 아래의 상황을 보자. 

```java
    static String[] food = {"김밥", "컵라면"};

    public static void main(String[] args) {
        Mountain[] mountains = {new Mountain("한라산"), new Mountain("설악산"), new Mountain("지리산"), new Mountain("소백산")};
            try {
                int i = 0;
                while (true) {
                    mountains[i].climb();
                    eat(i);
                    i++;
                }
            } catch (ArrayIndexOutOfBoundsException e) {

            }

    }

    static void eat(int i) {
        System.out.println(food[i] + "을 먹어요 !");
    }
```
try-catch문 외의 배열로 인한 ArrayIndexOutOfBoundsException 예외가 던져졌다. 즉, 의도하지 않은 ArrayIndexOutOfBoundsException이 던져져 내부의 버그가 발생한 것인데 프로그래머는 이것이 배열의 순회가 끝났다고 생각할 수 있다. 하지만 출력문을 보자. 

```
한라산 등산 시작 !
김밥을 먹어요 !
설악산 등산 시작 !
컵라면을 먹어요 !
지리산 등산 시작 !
```

산은 `한라산, 설악산, 지리산, 소백산` 까지 있는데, 지리산까지만 순회된 것으로 보인다. 즉, `eat()`메서드에서 ArrayIndexOutOfBoundsException예외가 던져져 배열의 순회가 제대로 이루어지지 못한 것이다. 

따라서 프로그래머는 이 버그를 제대로 잡기 어렵다. 

> 예외는 오직 예외 상황에서만 써야 한다. 절대로 일상적인 제어 흐름용으로 쓰여선 안된다. 

API도 마찬가지다. 잘 설계된 API라면 클라이언트가 정상적인 제어 흐름에서 예외를 사용할 일이 없게 해야한다. 

특정 상태에서만 호출할 수 있는 '상태 의존적' 메서드를 제공하는 클래스는 '상태 검사'메서드도 함께 제공해야한다. 

Iterator 인터페이스의 next와 hasNext가 각각 상태 의존적 메서드와 상태 검사 메서드에 해당한다. 

next는 다음 원소를 반환하는 메서드이기 때문에 원소가 있어야 호출할 수 있고, hasNext는 다음 원소가 있는지를 반환하는 메서드이기 때문이다. 아래를 보자. 
```java

        List<Mountain> mountains = List.of(new Mountain("한라산"), new Mountain("설악산"), new Mountain("지리산"), new Mountain("소백산"));
        for (Iterator<Mountain> i = mountains.iterator(); i.hasNext(); ) {
            Mountain mountain = i.next();
            mountain.climb();
        }
```
위와 같이 사용할 수 있다. 하지만, `hasNext`가 제공되지 않는다면 아래와 같이 작성해야할 것이다. 

```java
        try {
            Iterator<Mountain> i = mountains.iterator();
            while (true) {
                Mountain mountain = i.next();
                mountain.climb();
            }
        } catch (NoSuchElementException e) {

        }
```

상태 검사 메서드 대신 올바르지 않은 상태일 때 빈 옵셔널 혹은 null 같은 특수한 값을 반환하는 방법이다. 

상태 검사 메서드, 옵셔널, 특정 값 중 하나를 선택하는 지침을 살펴보자. 

1. 외부 동기화 없이 여러 스레드가 동시에 접근할 수 있거나 외부 요인으로 상태가 변할 수 있다면 옵셔널이나 특정 값을 사용한다. 상태 검사 메서드와 상태 의존적 메서드 호출 사이에 객체의 상태가 변할 수 있기 때문이다. 
2. 성능이 중요한 상황에서 상태 검사 메서드가 상태 의존적 메서드의 작업 일부를 중복 수행한다면 옵셔널이나 특정 값을 선택한다. 
3. 다른 모든 경우엔 상태 검사 메서드 방식이 조듬 더 낫다. 
   - 가독성, 버그 발견면에서 더 좋다. 
   
   
#### 출처

이펙티브 자바 3/E

