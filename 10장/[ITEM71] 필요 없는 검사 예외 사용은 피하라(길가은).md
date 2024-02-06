# 검사 예외
검사예외는 발생한 문제를 프로그래머가 처리하여 안전성을 높이게끔 해준다. 하지만 과하게 사용하면 오히려 쓰기 불편한 API가 된다. 

검사예외를 사용하면 try-catch문을 사용하거나, 강제로 바깥으로 예외를 던져야해 부담이 크다. 또한 검사 예외를 던지는 메서드는 스트림 안에서 직접 사용할 수 없다. 


API를 제대로 사용해도 발생할 수 있는 예외이거나 프로그래머가 의미있는 조치를 취할 수 있는 경우라면 이 정도 부담쯤은 받아들일 수 있을 것이지만 그렇지 않다면 비검사 예외를 사용하는 것이 좋다. 


```java
    public static void checkedError() {
        try {
            throw new CheckedException();
        } catch (CheckedException e) {
            throw new AssertionError();
        }
    }
    
    public static void checkedError() {
        try {
            throw new CheckedException();
        } catch (CheckedException e) {
            e.printStackTrace();
            System.exit(1);
        }
    }
```
위와 같은 방식을 사용할 수 있으나, 이는 최선의 방법이 아니다. 

검사 예외가 프로그래머에게 지우는 부담은 메서드가 단 하나의 검사예외만 던질 때가 특히 크다. 여러개라면 catch문 하나를 추가하면 되지만, 하나라면 이를 위해 try-catch문을 사용하고 스트림에선 사용도 못한다. 

## 옵셔널 반환
따라서 검사 예외를 피하기 위해 적절한 결과 타입을 담은 옵셔널을 반환하자. 이는 예외가 발생한 이유를 알려주는 부가정보를 담을 수 없지만 예외를 사용하면 구체적인 예외 타입과 그 타입이 제공하는 메서드들을 활용해 부가정보를 제공할 수 있다. 

다음과 같은 repository가 있다고 해보자. 
```java
public class Repository {

    private static final Map<Long, String> repository = new HashMap<>();
    private static Long id = -1L;

    public Long save(String value) {
        repository.put(++id, value);
        return id;
    }

    public String findById(Long id) throws CheckedException{
        String value = repository.get(id);
        if (value == null) {
            throw new CheckedException();
        }
        return value;
    }

}
```
위 repository는 조회했을 때 그 값이 null이면 검사 예외를 던진다. 이때 findById를 사용하는 클라이언트는 무조건 예외를 바깥으로 던지거나 try-catch 문을 사용해야만 한다. 

![](https://velog.velcdn.com/images/rlfrkdms1/post/5c99d2fa-08c4-4174-95e6-88239a0206d9/image.png)

![](https://velog.velcdn.com/images/rlfrkdms1/post/dc609442-c6d7-4b38-8bdc-57a1ece3c4fa/image.png)

따라서 이 메서드를 사용하는 클라이언트의 부담이 심하다. 이를 Optional로 바꿔보자. 

```java
    public Optional<String> findById(Long id){
        return Optional.of(repository.get(id));
    }
```
위와 같이 가독성도 좋아지고 코드도 짧아졌다. 

## 메서드 쪼개기
다른 방법으로는 검사 예외를 던지는 메서드를 2개로 쪼개 비검사 예외로 바꾸는 것이다. 기존 repository 코드를 살펴보자. 

```java
    public String findById(Long id) throws CheckedException{
        String value = repository.get(id);
        if (value == null) {
            throw new CheckedException();
        }
        return value;
    }
```
이는 아래와 같이 사용했다. 

```java
    public static void main(String[] args) {
        Repository repository = new Repository();
        try {
            String value = repository.findById(2L);
        } catch (CheckedException e) {
            throw new RuntimeException(e);
        }
    }
```

findById를 쪼개보자. 

```java
    public String findById(Long id) {
        return repository.get(id);
    }

    public boolean existById(Long id) {
        return repository.containsKey(id);
    }
```
그러면 아래와 같이 사용할 수 있다. 

```java
    public static void main(String[] args) {
        Repository repository = new Repository();
        if (repository.existById(2L)) {
            repository.findById(2L);
        } else {
            //예외 상황 처리
        }
    }
```

이 리팩터링을 모든 상황에 적용할 순 없지만 적용할 수만 있다면 더 쓰기 편한 API를 제공할 수 있다. 하지만 외부 동기화 없이 여러 스레드가 동시에 접근할 수 있거나 외부 요인에 의해 상태가 변할 수 있다면 이 리팩터링은 적절하지 않다. existById()와 findById() 사이에 객체의 상태가 변할 수 있기 때문이다. 

또한 작업이 일부 중복 수행된다면 성능상에서도 손해이다. 

#### 출처

이펙티브 자바 3/E

