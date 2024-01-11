하나 이상에 자원에 의존하며, 의존하는 자원에 따라 동작이 달라지는 클래스를 정적 유틸리티 클래스나 싱글턴 클래스로 구현하는 경우가 많다. 
```java
public class Robot {
    private final Arm arm = new Arm();

    private Robot() {
        throw new AssertionError();
    }
}
```
```java
public class Robot {
    private final Arm arm = new Arm();
    private final static Robot INSTANCE = new Robot();

    private Robot() {
        throw new AssertionError();
    }
}
```
- 정적 유틸리티 클래스나 싱글턴 방식으로 Robot 클래스를 정의하면, Arm 객체를 변경할 수 없다. 하나의 Arm 객체로 모든 상황에 대응해야 한다. 
- 유연하지 않고 테스트하기 어렵다.

- arm 필드에서 final을 제거하고 Arm을 교체하는 메서드를 추가해서 Arm을 변경할 수 있다.
  - Arm의 불변을 보장하지 않는다. 
  - 어색하고 오류를 내기 쉬우며 멀티스레드 환경에서 쓸 수 없다. 

사용하는 자원에 따라 동작이 달라지는 클래스에스에는 정적 유틸리티 클래스나 싱글턴 방식이 부적절하다. 

# 의존 객체 주입 
의존 객체 주입은 인스턴스를 생성할 때 생성자에 필요한 자원을 넘겨주는 것이다.
```java
public class Robot {
    private final Arm arm;

    public Robot(Arm arm) {
        this.arm = arm;
    }
}
```
- 의존 객체(자원)의 불변을 보장하여 여러 클라이언트가 안심하고 공유할 수 있다.
- 생성자, 정적 팩터리, 빌더에 응용할 수 있다.

# 팩터리 메서드 패턴 
의존 객체 주입의 변형으로 생성자에 자원 팩터리를 넘겨준다. 
> 팩터리 : 호출할 때 마다 특정 타입의 인스턴스를 반복해서 만들어주는 객체
```java
public class Robot {
    private final Arm arm;
    private final Factory<Arm> factory;

    //생성자에 자원 팩터리를 넘겨준다
    public Robot(Factory<Arm> factory) {
        this.factory = factory;
        this.arm = factory.create();
    }
}


// 자원 인터페이스
public interface Arm {
    public void attack();
}

// 자원 구현체
public class GunArm implements Arm{
    @Override
    public void attack() {
        System.out.println("gun attack");
    }
}

// 자원 구현체
public class SwordArm implements Arm{
    @Override
    public void attack() {
        System.out.println("sword attack");
    }
}

// 자원 팩토리
public abstract class Factory<T extends Arm> {
    public abstract T create();
}

// 자원 팩토리 구현체
public class GunArmFactory extends Factory<Arm> {
    @Override
    public GunArm create() {
        return new GunArm();
    }
}

// 자원 팩토리 구현체
public class SwordArmFactory extends Factory<Arm> {
    @Override
    public SwordArm create() {
        return new SwordArm();
    }
}

// 클라이언트 코드
    public static void main(String[] args) {
        Robot robot = new Robot(new GunArmFactory());
    }
```
Robot은 자원(Arm)의 팩터리(Factory)를 생성자에서 전달받고 팩터리를 통해 자원을 생성한다. 
팩터리의 `create()` 메서드 반환형은 자원의 추상 인터페이스 Arm으로, Factory를 구현한 구현체에 따라 다른 자원 구현체를 반환한다. 

# 의존 객체 주입 단점
- 의존성이 많은(수천개) 프로젝트에서는 코드를 어지럽게 만든다.
- 대거, 주스, 스프링같은 DI 프레임워크를 사용하면 어질러짐을 해소할 수 있다.

  







