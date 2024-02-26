java.util.concurrent 패키지를 통해 인터페이스 기반의 유연한 태스크 실행기능을 사용할 수 있다. 

작업 큐를 아래와 같이 생성할 수 있다. 
```java
ExecutorService exec = Executors.newSingleThreadExecutor();
```
이를 `exec.execute(runnable);`과 같이 실행자에 실행할 태스트를 넘길 수 있다. 또한 `exec.shutdown()`으로 종료시킬 수도 있다. 

실행자 서비스의 주요 기능들을 살펴보자. 
- 특정 태스크가 완료되기를 기다린다. 
- 태스크 모음 중 아무것 하나(`invokeAny`) 혹은 모든 태스크(`invokeAll`)가 완료되기를 기다린다. 
- 실행자 서비스가 종료하기를 기다린다.(`awaitTermination`)
- 완료된 태스크들의 결과를 차례로 받는다. (`ExecutorCompleteService`)
- 태스크를 특정 시간에 혹은 주기적으로 실행하게 한다. (`ScheduledThreadPoolExecutor`)

실행자 서비스를 사용하기에 까다로운 애플리케이션도 있다. 

- 작은 프로그램이나 가벼운 서버 
   - `Executors.newCachedThreadPool` : 요청 받은 태스크들이 큐에 쌓이지 않고 즉시 스레드에 위임
- 무거운 프로덕션 서버 
   - `Executors.newFixedThreadPool` : 스레드 개수 고정
   - `ThreadPoolExecutor` : 완전 통제 가능

작업 큐를 손수 만드는 일을 삼가고 스레드를 직접 다루는 것도 일반족으로 삼가자. 

- Task : 작업 단위를 나타내는 핵심 추상 개념
   - Runnable
   - Callable
   - 태스크 수행 -> 실행자 서비스
      - 실행자 서비스를 통해 태스크 수행 정책 선택 가능
      - 실행자 프레임 워크가 작업 수행을 담당
   
실행자 프레임 워크
- ForkJoinTask
   - 인스턴스는 작은 하위 태스크로 나뉠 수 있음
   - ForkJoinPool 구성 스레드들이 이 태스크들을 처리
   - 일을 먼저 끝낸 스레드는 다른 스레드의 남은 태스크 대신 처리
   - ForkJoinPool을 이용해 만든 병렬 스트림 이용
   
#### 출처

이펙티브 자바 3/E
