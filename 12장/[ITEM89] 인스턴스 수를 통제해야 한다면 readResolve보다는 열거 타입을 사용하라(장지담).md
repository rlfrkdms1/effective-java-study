```java
public class Robot {
    public static final Robot INSTANCE = new Robot();

    private Robot() {
        // ... 
    }
}
```
- 위와 같은 싱글턴 클래스에 implements Serializable을 추가하면 싱글턴이 아니게 된다.
- 기본 직렬화를 쓰지 않거나 readObject를 명시해도 소용없다. 어떤 readObejct든 클래스가 초기화될 때 만들어진 INSTANCE와 다른 인스턴스를 반환하게 된다.

# readResolve
```java
public class Robot implements Serializable {
    public static final Robot INSTANCE = new Robot();

    private Robot() {
        // ...
    }

    private Object readResolve() {
        return INSTANCE;
    }
}
```
- 클래스에 readResolve 메서드를 정의하면 역직렬화 할 때 이 메서드가 반환하는 객체 참조가 대신 반환된다. 새로 생성된 객체는 바로 가비지 컬렉션 대상이 된다.
- 이런 식으로 readResolve를 정의하면 Robot 인스턴스의 직렬화 형태는 아무런 실 데이터를 가질 필요가 없으니 모든 객체 참조 필드를 transient로 선언해야 한다. 그렇지 않으면 ITEM88에서 소개한 것 같은 공격을 받을 수 있다.

### 공격에 관해 알아보자
- 싱글턴이 transient가 아닌 참조 필드를 갖고 있다면 그 필드의 내용은 readResolve가 실행되기 전에 역직렬화된다. 잘 조작된 스트림을 써서 해당 참조 필드의 내용이 역직렬화되는 시점에 역직렬화된 인스턴스의 참조를 훔쳐올 수 있다. 
```java
public class Robot implements Serializable {

    private Arm arm;
    public static final Robot INSTANCE = new Robot();

    private Robot() {
        // ...
    }

    private Object readResolve() {
        return INSTANCE;
    }
}
```
- 위 Robot 클래스는 transient가 아닌 참조 필드를 갖고 있다.
```java
public class RobotStealer {
    static Robot impersonator;
    private Robot payload;

    private Object readResolve() {
        impersonator = payload;

        return new Arm();
    }
}
```
- readResolve 메서드와 인스턴스 필드를 포함하는 도둑 클래스를 만들 수 있다 
- 인스턴스 필드는 숨길 직렬화된 싱글턴을 참조하는 역할이다. (위에서는 Robot)
```java
public class RobotImpersonator {
    // 진짜 Robot 인스턴스로는 만들어질 수 없는 바이트 스트림
	private static final byte[] serializedForm = {
            -84, -19, 0, 5, 115, 114, 0, 20, 107, 114, 46, 115,
            101, 111, 107, 46, 105, 116, 101, 109, 56, 57, 46, 69,
            108, 118, 105, 115, 98, -14, -118, -33, -113, -3, -32, 
		    70, 2, 0, 1, 91, 0, 13, 102, 97, 118, 111, 114, 105, 116, 
		    101, 83, 111, 110, 103, 115, 116, 0, 19, 91, 76, 106, 97, 
		    118, 97, 47, 108, 97, 110, 103, 47, 83, 116, 114, 105, 110, 
		    103, 59, 120, 112, 117, 114, 0, 19, 91, 76, 106, 97, 118, 
		    97, 46, 108, 97, 110, 103, 46, 83, 116, 114, 105, 110, 103, 
		    59, -83, -46, 86, -25, -23, 29, 123, 71, 2, 0, 0, 120, 112, 
		    0, 0, 0, 2, 116, 0, 9, 72, 111, 117, 110, 100, 32, 68, 111, 
		    103, 116, 0, 16, 72, 101, 97, 114, 116, 98, 114, 101, 97, 107, 
		    32, 72, 111, 116, 101, 108
    };
    public static void main(String[] args) {
        Robot robot = (Robot) deserialize(serializedForm);
        Robot impersonator = Robot.impersonator;
    }
}
```
- 직렬화된 스트림에서 싱글턴의 비휘발성 필드를 도둑 인스턴스로 교체한다.
- 싱글턴은 도둑을 참조하고 도둑은 싱글턴을 참조하게 된다. 싱글턴이 역직렬화 될 때 도둑의 readResolve를 먼저 호출한다. 도둑의 readResolve가 수행될 때 도둑의 인슽언스 필드에는 역직렬화 도중인 (readResolve가 수행되기 전인) 싱글턴의 참조가 있게 된다.
- 위와 같은 방법으로 서로 다른 2개의 Robot 인스터스를 생성할 수 있다.

---

# 원소 하나짜리 열거 타입 사용 

- 이러한 공격은 Robot의 Arm 필드를 transient로 선언하여 고칠 수 있다. 하지만 Robot을 원소 하나짜리 열거 타입으로 바꾼느 편이 낫다. 
- 열거타입을 활용해 직렬화 가능한 인스턴스 통제 타입을 사용하면 자바가 선언한 상수 외의 다른 객체는 존재하지 않음을 보장한다.
- AccesibleObject.setAccessible 같은 특권 메서드와 같은 공격에는 안전하지 않음 

```java
public enum Robot {
    INSTANCE;
    private Arm arm = new Arm(10);

    public void punch() {
        arm.punch();
    }
}
```

# readResolve 를 사용하는 경우
- 직렬화 가능 인스턴스 통제 클래스를 작성해야 하는데 컴파일타임에는 어떤 인스턴스들이 있는지 알 수 없는 상황에는 열거 타입을 사용할 수 없다.
- final 클래스에서는 readResolve는 private이어야 한다.
- final이 아닌 클래스에서는
  - private으로 선언하면 하위 클래스에서 사용할 수 없다
  - package priavte으로 선언하면 같은 패키지에 속한 하위 클래스에서만 사용할 수 있다.
  - protected, public으로 선언하면 재정의하지 않은 모든 하위 클래스에서 사용할 수 있다. 이 경우 ClassCastException을 일으킬 수 있다. 
