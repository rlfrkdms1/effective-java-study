# 공통 상위 클래스 하나로 뭉뚱그려 예외선언을 하지 말자 
```java
    public void punch() throws Exception {
        // do something
    }
```
- 공통 상위 클래스 Exception 으로 뭉뚱그려 예외선언했다.
- 다른 예외들까지 삼켜버려 API 활용성이 떨어진다
- 메서드 사용자에게 각 예외에 대처할 수 있는 힌트를 주지 못한다

예외적으로 main은 JVM만이 호출하므로 무관하다. (Exception을 던지도록 선언해도 괜찮다)

# 비검사 예외도 문서화해두면 좋다
```java
    /**
     * @throws CheckedException   어쩌구저쩌구 경우
     * @throws UnCheckedException 저쩌구어쩌구 경우
     */
    public void move() throws CheckedException {
        // do something
    }
```
- @throws로 문서화는 하되, 비검사 예외는 throws 목록에 넣지 말자. 이렇게 작성하면 javadoc이 시각적으로 구분해준다. 
- 검사, 비검사 여부에 따라 API 사용자가 할 일이 달라지기 때문

![image](https://github.com/rlfrkdms1/effective-java-study/assets/96513365/8c03583d-c629-494b-9682-07af2756c71f)

- 직접 javadoc을 만들어 봤다. 메서드 선언에 포함한 예외와 @throws만 사용한 예외를 구분할 수 있다.
- 다만 모든 비검사 예외를 문서화하는 것은 현실적으로 불가능 할 때도 있다.

# 클래스 설명에 예외를 추가할 수 있다
```java
/**
 * {@code CheckedException} 이 클래스의 모든 메서드는 ~ 경우에 CheckedEception을 던집니다. 
 */
public class Robot {
...
}
```
- 한 클래스에 같은 원인으로 인해 같은 예외를 던지는 메서드가 많으면 클래스에 문서화 주석을 추가할 수 있다.
- 참고 : `{@code}`는 해당 부분을 javadoc 문서에서 코드 형식으로 나타내기 위함이다.  

