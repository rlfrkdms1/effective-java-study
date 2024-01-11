싱글턴은 인스턴스를 오직 하나만 생성할 수 있는 클래스이다.
클래스를 싱글턴으로 만드는 3가지 방법이 있다.
#  public static final 필드 
```java
public class Robot {
    
    public static final Robot INSTANCE = new Robot();
    private Robot(){
        ..
    }
}
```
- private 생성자는 INSTANCE를 초기화 할 때 한번만 호출된다.
- 다른 생성자가 없어서 싱글턴이 보장된다
- 예외적으로 권한이 있는 클라이언트는 리플렉션 API를 통해 private 생성자를 호출할 수 있다. 이러한 공격을 막기 위해 두번째 객체가 생성될 때 예외를 던진다.
- api에 싱글턴임이 명확히 드러난다.
- 간결하다

# 정적 팩터리 메서드 
```java
public class Robot {

    public static final Robot INSTANCE = new Robot();
    public static Robot getInstance(){return INSTANCE;}
    private Robot(){
        ..
    }
}
```
- `getInstance()`는 항상 같은 참조를 반환하므로 싱글턴이 보장된다. (리플렉션 API에 의한 예외는 발생할 수 있다)
- api를 바꾸지 않고도 getInstance()가 다른 객체를 반환하게 변경해서 싱글턴이 아니게 변경할 수 있다.
- 정적 팩터리를 제네릭 싱글턴 팩터리로 만들 수 있다. (제네릭 싱글턴 팩터리(아이템30) : 제네릭으로 타입설정 가능한 인스턴스를 만들어두고, 반환 시에 제네릭으로 받은 타입을 이용해 타입을 결정하는 것)
- 정적 팩토리 참조를 공급자로 사용할 수 있다(아이템43,44).

---

위에서 소개한 2가지 방식으로 만든 싱글턴 클래스를 직렬화하는 것은 Serializable을 구현하는 것 만으로 부족하다. 
모든 인스턴스를 transient라고 선언하고 readResolve 메서드를 제공해야 한다(아이템89). 
이렇게 하지 않으면 직렬화된 객체를 역직렬화 할 때마다 새로운 객체가 만들어진다.

> 직렬화 : 자바 시스템 내부에서 사용되는 객체 또는 데이터를 외부의 자바 시스템에서도 사용할 수 있도록 바이트(byte) 형태로 데이터 변환하는 기술. Serializable 인터페이스를 구현해야 직렬화 할 수 있다.

# 원소가 하나인 열거타입 선언 
```java
public enum Robot {
    INSTANCE(10);

    Robot(int battery) {
        this.battery = battery;
    }

    private int battery;

    public void move(){
        
    }
}
```
- 열거형 값은 해당 자료형의 인스턴스이다.
- 원소가 하나인 열거타입을 선언해서 싱글턴으로 사용한다
- public 방식보다 간결하다
- 직렬화하기 쉽다. enum은 기본적으로 serializable 하다. 결과로 constant의 이름을 얻는다.
- 만드려는 싱글턴이 enum 외 다른 클래스를 상속해야 한다면 사용할 수 없다. (인터페이스는 가능하다)


 

 

