여러 스레드가 실행 중이면 운영체제의 스레드 스케줄러가 어떤 스레드를 얼마나 오래 실행할지 정한다. 

> 정확성이나 성능이 스레드 스케줄러에 따라 달라지는 프로그램이라면 다른 플랫폼에 이식하기 어렵다. 

견고하고 빠릿하고 이식성 좋은 프로그램을 작성하는 가장 좋은 방법은 실행 가능한 스레드의 평균적인 수를 프로세서 수보다 지나치게 많아지지 않도록 하는 것이다. 

실행 가능한 스레드 수를 적게 유지하는 주요 기법은 각 스레드가 무언가 유용한 작업을 완료한 후에는 다음 일거리가 생길 때까지 대기하도록 하는 것이다. 

> 스레드는 당장 처리해야할 작업이 없다면 실행돼서는 안된다. 

스레드는 절대 공유 객체의 상태가 바뀔 때까지 쉬지 않고 검사(바쁜 대기)해서는 안된다. 이는 스케줄러의 변덕에 취약하고, 프로세서에 큰 부담을 주어 다른 유용한 작업이 실행될 기회를 박탈한다. 

```java
public class SlowCountDownLatch {
    private int count;

    public SlowCountDownLatch(int count) {
        if (count < 0)
            throw new IllegalArgumentException(count + " < 0");
        this.count = count;
    }

    public void await() {
        while (true) {
            synchronized(this) {
                if (count == 0)
                    return;
            }
        }
    }
    public synchronized void countDown() {
        if (count != 0)
            count--;
    }
}
```
하나 이상의 스레드가 필요도 없이 실행가능한 상태이다. 

Thread.yield를 써서 문제를 고치려 들진 말자. 증상이 어느 정도는 호전될 수도 있지만, 이식성은 그렇지 않을 것이다. 테스트할 수단도 없다. 

차라리 애플리케이션 구조를 바꿔 동시에 실행 가능한 스레드 수가 적어지도록 조치해주자. 

#### 출처

이펙티브 자바 3/E
