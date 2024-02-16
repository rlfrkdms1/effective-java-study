# 동기화
대부분의 프로그래머들은 동기화를 한 스레드가 변경 중이라 상태가 일관되지 않은 순간의 객체를 다른 스레드가 보지 못하게 막는 용도로만 생각한다. 

즉, 객체를 하나의 일관된 상태에서 다른 일관된 상태로 변화시키는 것인데 중요한 기능이 하나 더 있다. 동기화의 기능은 아래 두가지로 나눌 수 있다. 

1. 일관성이 깨진 상태를 볼 수 없게 함
2. 동기화된 메서드나 블록에 들어간 스레드가 같은 락의 보호하에 수행된 모든 이전 수정의 최종 결과를 보게 함

## 원자적 데이터

언어 명세상 long, double외의 변수를 읽고 쓰는 동작은 원자적이다. 이 말은 여러 스레드가 같은 변수를 동기화 없이 수정하는 중이라도, 항상 어떤 스레드가 정상적으로 저장한 값을 온전히 읽어옴을 보장한다는 것이다. 

이때 주의해야할 점이 있다. 자바는 스레드가 필드를 읽을 때 항상 **수정이 완전히 반영된** 값을 **얻는다**고 보장하지만, 한 스레드가 저장한 값이 다른 스레드에게 **보이는가**는 보장하지 않는다. 

따라서 동기화는 스레드 사이의 안정적인 통신을 위해서도 꼭 필요한 존재다. 

### 스레드를 멈추는 작업

#### Thread.stop

위의 메서드는 안전하지 않으니, 사용하지 말자. 

#### boolean 필드를 활용

아래의 코드를 살펴보자. 
```java
public class StopThread {
    private static boolean stopRequested;

    public static void main(String[] args)
            throws InterruptedException {
        Thread backgroundThread = new Thread(() -> {
            int i = 0;
            while (!stopRequested)
                i++;
        });
        backgroundThread.start();

        TimeUnit.SECONDS.sleep(1);
        stopRequested = true;
    }
}
```
while문의 `stopRequested`를 통해 반복문을 제어하고 있다. 

```java
        TimeUnit.SECONDS.sleep(1);
        stopRequested = true;
```
을 통해서 1초뒤에 while문이 멈출 것이라고 생각하지만, 아니다. 왜일까? 동기화 때문이다. 

동기화를 하지 않았기 때문에 가상 머신이 다음과 같은 최적화를 수행할 수 있다. 이는 끌어올리기라는 최적화 기법이다. 

```java
            while (!stopRequested)
                i++;
```
위와 같으나, 이는 아래와 같이 최적화된다. 
```java
		if(!stopRequested)
            while (!stopRequested)
                i++;
```
따라서 stopReqeusted의 값이 바뀌어도 while문은 멈추지 않는것이다. 따라서 동기화를 사용해 아래와 같이 변경하자. 

### synchronized

```java
public class StopThread {
    private static boolean stopRequested;

    private static synchronized void requestStop() {
        stopRequested = true;
    }

    private static synchronized boolean stopRequested() {
        return stopRequested;
    }

    public static void main(String[] args)
            throws InterruptedException {
        Thread backgroundThread = new Thread(() -> {
            int i = 0;
            while (!stopRequested())
                i++;
        });
        backgroundThread.start();

        TimeUnit.SECONDS.sleep(1);
        requestStop();
    }
}  
```
위처럼 stopRequested 필드를 true로 변환하고, 읽어오는 메서드 모두 동기화를 해주었다. 여기서 stopRequested를 true로 변환하는 requestStop()메서드만 동기화하면 될 것 같지만, 이것만 동기화해서는 위에서와 같이 반복문을 멈추지 않을 것이다. 

따라서 쓰기와 읽기 모두가 동기화되지 않으면 동작을 보장하지 않음을 기억하자 !

### volatile
다른 대안이 또 존재한다. 바로 volatile이다. 필드를 volatile로 선언하면 동기화를 생략해도 된다. 이는 배타적 수행과는 상관없지만 항상 최근에 기록된 값을 읽게 됨을 보장한다. 
```java
public class StopThread {
    private static volatile boolean stopRequested;

    public static void main(String[] args)
            throws InterruptedException {
        Thread backgroundThread = new Thread(() -> {
            int i = 0;
            while (!stopRequested)
                i++;
        });
        backgroundThread.start();

        TimeUnit.SECONDS.sleep(1);
        stopRequested = true;
    }
}
```

하지만 volatile는 주의해서 사용해야한다. 다음의 예제를 살펴보자. 

```java
    private static volatile int nextSerialNumber = 0;
    
    public static int generateSerialNumber(){
        return nextSerialNumber++;
    }
```
위의 generateSerialNumber메서드는 nextSerialNumber를 반환하는 메서드이다. 아무런 문제가 없어보이지만, 증가연산자 때문에 올바르게 동작하지 않는다. 증가 연산자는 값을 읽을 때 한 번, 값을 저장할 때 한 번으로 총 두번 필드에 접근한다. 따라서 다음과 같은 상황이라면 문제가 생기는 것이다. 

nextSerialNumber는 현재 0이며, 1번과 2번 스레드가 돌아간다고 가정하자. 

1. 1번 스레드가 nextSerialNumber를 읽는다. 
2. 2번 스레드가 nextSerialNumber를 읽는다. 
3. 1번 스레드가 1을 더한 값을 저장한다. 즉, nextSerialNumber는 1이된다. 
4. 2번 스레드가 읽은 값에서 1을 더한 값을 저장한다. 즉, nextSerialNumber는 1이된다. 

위와 같이 첫번째 스레드가 연산을 진행하고 있는 사이에 두번째 스레드가 접근한다면, 올바른 결과값이 나오지 않게되는 것이다. 이렇게 프로그램이 잘못된 결과를 계산해내는 오류를 안전 실패라고 한다. 

이는 synchronized를 메서드에 붙이면 해결된다. synchronized를 붙였다면 필드에 있는 volatile은 제거해야한다. 

## AtomicLong

락 없이도 스레드 안전한 프로그래밍을 지원하는 클래스가 담긴 AtomicLong을 사용해보자. 

```java
    private static final AtomicLong nextSerialNumber = new AtomicLong();

    public static long generateSerialNumber(){
        return nextSerialNumber.getAndIncrement();
    }
```
이는 원자성과 통신 모두를 지원한다. 

## 가장 좋은 방법
> 이러한 문제들을 피하는 가장 좋은 방법은 가변 데이터를 공유하지 않는것이다. 

따라서 가변 데이터는 단일 스레드에서만 쓰도록 하자. 


한 스레드가 데이터를 다 수정한 후 다른 스레드에 공유할 땐 해당 객체에서 공유하는 부분만 동기화해도 된다. 이러면 수정전까지 다른 스레드들이 동기화없이 값을 읽어갈 수 있다. 이런 객체를 사실상 불변이라 하고, 다른 스레드에 이런 객체를 건네는 행위를 안전발행이라고 한다. 

객체를 안전하게 발행하는 방법
- 클래스 초기화 과정에서 객체를 정적 필드, volatile 필드, final 필드, 보통의 락을 통해 접근하는 필드에 저장
- 동시성 컬렉션에 저장

#### 출처

이펙티브 자바 3/E
