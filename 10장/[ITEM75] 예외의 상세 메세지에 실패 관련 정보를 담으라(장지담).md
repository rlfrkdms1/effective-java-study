# 예외의 toString이 실패 원인에 관한 정보를 최대한 많이 담아 반환해야 한다. 
```java
    public String toString() {
        String s = getClass().getName();
        String message = getLocalizedMessage();
        return (message != null) ? (s + ": " + message) : s;
    }
```
- 위 예시는 Throwable의 toString이다. 예외의 toString은 오버라이딩하지 않으면 Throwable의 toString이며 이것은 `예외명 : 예외메세지` 형태다
- 프로그램이 예외로 인해 실패하면 스택 추적 정보를 출력한다. 스택 추적 정보는 예외객체의 toString을 호출해 얻는 문자열을 보여준다.

### 예외에 관여된 모든 매개변수와 필드의 값을 실패 메세지에 담아야 한다
```java
    public int charge(int fuel) {
        if (fuel <= 0 || energy + fuel <= 0) {
            throw new IllegalArgumentException(
                    String.format("energy : %d, fuel : %d", energy, fuel));
        }
        energy += fuel;
        return energy;
    }
```
- 매개변수 fuel과 필드 energy가 예외에 관여되어 있다. 따라서 생성자에 energy와 fuel을 포함한 예외 메세지를 전달한다.
  - 표준 예외 IllegalArguemtnException을 재사용하므로 toString을 오버라이딩하거나 별도의 메서드, 필드를 추가할 수 없다. 따라서 예외메세지에 출력하고자 하는 내용을 담아야 한다.
- 예외 원인 분석과 문제 해결에 도움이 된다.
- 문서, 소스코드에서 얻을 수 있는 정보(줄번호 등)은 불필요하다.
  - 프로그래머를 대상으로 하는 예외 메세지는 가독성보다 담긴 내용이 중요하다. 

![image](https://github.com/rlfrkdms1/effective-java-study/assets/96513365/b9a69525-ad3c-4a55-b1e9-f7ff9a7bccea)

스택 추적 정보에서 전달한 메세지를 보여준다. 

### 필요한 정보를 예외 생성자에서 받아 상태 메세지를 생성해놓자
```java
    public RunException(int start, int end, int speed) {
        super(String.format("시작위치 : %d, 끝위치 : %d, 속도 : %d", start, end, speed));
        this.start = start;
        this.end = end;
        this.speed = speed;
    }

    public int getStart() {
        return start;
    }
```
- 예외 생성자에서 필요한 정보들을 매개변수로 받아 예외 메세지를 생성한다.
- 실패를 포착하기 쉽고, 예외 메세지를 만드는 작업이 예외 클래스 안으로 모아져 중복이 감소한다. 
- 필요한 정보들을 예외 필드로 저장하므로 getter로 제공할 수 있다. 예외와 관련된 정보를 얻을 수 있는 getter를 제공하는 것이 좋다. 
