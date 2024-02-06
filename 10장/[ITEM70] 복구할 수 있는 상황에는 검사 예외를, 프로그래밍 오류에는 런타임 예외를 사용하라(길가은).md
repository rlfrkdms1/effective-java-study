자바는 검사 예외, 런타임 예외, 에러 세가지를 제공하는데 언제 무엇을 사용해야하는지 알아보자. 

![](https://velog.velcdn.com/images/rlfrkdms1/post/6ecaf6c7-6d27-4816-b7f6-85d2c593258c/image.png)


> 호출하는 쪽에서 복구하리라 여겨지는 상황이라면 검사예외를 사용하라

검사 예외를 던지면 호출자가 그 예외를 catch로 잡아 처리하거나 더 바깥으로 전파하도록 강제하게된다. 따라서 그 메서드를 호출했을 때 발생할 수 있는 유력한 결과임을 API 사용자에게 알려준다. 

![](https://velog.velcdn.com/images/rlfrkdms1/post/554acf8e-ae9b-4d12-8cc1-a02a098c0f4a/image.png)
위와 같이 검사예외인 IOException은 던지면 컴파일 에러가 뜬다. 에러의 내용은 다음과 같다. 

```
Unhandled exception: java.io.IOException
```
이는 예외를 바깥으로 던지거나 try-catch문을 사용하면 해결된다. 

```java
    public void checkedError() throws IOException {
        throw new IOException();
    }
    
    public void checkedError() {
        try {
            throw new IOException();
        } catch (IOException e) {
            throw new RuntimeException(e);
        }
    }
```

런타임 예외와 에러는 프로그램에서 잡을 필요가 없거나 통상적으로는 잡지 말아야 한다. 

> 프로그래밍 오류를 나타낼 때는 런타임 예외를 사용하자. 

런타임 예외의 대부분은 클라이언트가 해당 API의 명세에 기록된 제약을 지키지 못했을 때 발생한다. 

예로 아래와 같이 배열의 크기를 넘어선 접근이 있을 때 던져지는 ArrayIndexOutOfBoundsException이 있다. 

```java
    public static void main(String[] args) {
        String[] test = {"test"};
        test[1] = "hi";
    }
```
아래와 같은 에러가 던져 진다. 

```
Exception in thread "main" java.lang.ArrayIndexOutOfBoundsException: Index 1 out of bounds for length 1
	at chap10.item70.Test.main(Test.java:9)
```
하지만 이는 문제가 하나있다. 복구할 수 있는 상황인지 프로그래밍 오류인지 항상 명확히 구분되지 않는 것이다. 예로 자원 고갈일 때 말도 안되는 크기의 배열을 할당해 생긴 프로그래밍 오류인지, 진짜 자원이 부족해 생긴 문제인지 알 수 없다는 것이다. 

따라서 복구 가능하다고 믿는다면 검사 예외를, 그렇지 않다면 런타임 예외를 사용하자. 

에러는 보통 JVM이 자원 부족, 불변식 깨짐 등 더 이상 수행을 계속할 수 없는 상황을 나타낼 때 사용한다. 따라서 Error를 상속하지 말고, 구현하는 비검사 throwable은 모두 RuntimeException의 하위클래스로 만들자. 

참고로 Exception, RuntimeException, Error를 상속하지 않는 throwable은 이로운것도 없는데 API 사용자를 헷갈리게 하니 사용하지 말자. 


#### 출처
[throwable](https://www.whitman.edu/mathematics/java_tutorial/java/exceptions/throwable.html)

이펙티브 자바 3/E
