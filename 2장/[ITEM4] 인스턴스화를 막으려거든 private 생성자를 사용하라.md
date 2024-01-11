> 유틸리티 클래스 : 정적 필드, 정적 메서드만으로 구성된 클래스
```java
public class Writer {

    private static final String START_MENT = "숫자 야구 게임을 시작합니다";
    private static final String INPUT_MENT = "숫자를 입력해주세요 : ";
    private static final String CORRECT_MENT = "3개의 숫자를 모두 맞히셨습니다! 게임 종료";
    private static final String RESTART_MENT = "게임을 새로 시작하려면 1, 종료하려면 2를 입력하세요.";

    private Writer() {
        throw new UnsupportedOperationException();
    }

    public static void writeStartMent() {
        System.out.println(START_MENT);
    }

    public static void writeInputMent() {
        System.out.print(INPUT_MENT);
    }

    public static void writeGradeResult(GradeResult gradeResult) {
        System.out.println(gradeResult);
    }

    public static void writeCorrectMent() {
        System.out.println(CORRECT_MENT);
    }

    public static void writeRestartMent() {
        System.out.println(RESTART_MENT);
    }

}
```
- 유틸리티 클래스는 인스턴스 생성을 위한 클래스가 아니다.
- 생성자를 명시하지 않으면 public 기본 생성가 만들어져 유틸리티 클래스의 객체를 생성할 수 있다.
- 추상 클래스로 선언하면 상속해서 객체를 생성하면 된다. 또 사용자는 추상클래스를 상속하라는 의미로 받아들일 수 있다.

# private 생성자를 사용하자
```java
    // 객체 생성을 막기 위함 
    private Robot(){
        throw new AssertionError();
    }
```
- 생성자를 private으로 선언하면 컴파일러가 기본 생성자를 만들지 않는다
- 클래스 안에서 실수로라도 생성자를 호출하는 것을 방지하기 위해 AssertionError를 던진다
- 생성자가 존재하는데 호출할 수 없으므로 직관적이지 않다
  - 적절한 주석을 달아준다
- 상속을 불가능하게 한다
  - 모든 생성자는 상위 클래스의 생성자를 호출하는데, private이므로 호출할 수 없다. 

