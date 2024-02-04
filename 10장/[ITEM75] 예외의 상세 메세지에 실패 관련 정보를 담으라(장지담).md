# 예외의 toString이 실패 원인에 관한 정보를 최대한 많이 담아 반환해야 한다. 
```java
    public String toString() {
        String s = getClass().getName();
        String message = getLocalizedMessage();
        return (message != null) ? (s + ": " + message) : s;
    }
```
- 예외의 toString은 오버라이딩하지 않으면 Throwable의 toString이며 이것은 `예외명 : 예외메세지` 형태다
- 프로그램이 예외로 인해 실패하면 스택 추적 정보를 출력한다. 스택 추적 정보는 예외객체의 toString을 호출해 얻는 문자열을 보여준다.

