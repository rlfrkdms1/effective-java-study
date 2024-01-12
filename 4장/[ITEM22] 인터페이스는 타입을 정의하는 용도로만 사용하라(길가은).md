클래스가 어떤 인터페이스를 구현한다는 것은 자신의 인스턴스로 **무엇을 할 수 있는지**를 클라이언트에 얘기해주는 것이다. 오직 이 용도로만 사용해야한다. 

### 인터페이스의 잘못된 예, 상수 인터페이스
메서드 없이, 상수를 뜻하는 static final 필드로만 가득 찬 인터페이스이다. 

```java
public interface PhysicalConstants {
    // 아보가드로 수 (1/몰)
    static final double AVOGADROS_NUMBER   = 6.022_140_857e23;

    // 볼츠만 상수 (J/K)
    static final double BOLTZMANN_CONSTANT = 1.380_648_52e-23;

    // 전자 질량 (kg)
    static final double ELECTRON_MASS      = 9.109_383_56e-31;
}
```

이는 인터페이스를 잘못 사용한 예다. 

- 상수는 내부 구현에 해당 -> 내부 구현을 API로 노출하는 행위
    - private, protected로 선언하지 못하기 때문
- 클라이언트의 코드가 상수에 종속되게 함 -> 다음 릴리스에서 상수를 쓰지 않아도 호환성을 위해 여전히 구현해야함

상수를 공개할 목적이라면 다음의 선택지가 있다. 

1. 특정 클래스나, 인터페이스와 강하게 연관 -> 클래스나 인터페이스 자체에 추가
```java 
public class PhysicalConstants {
  private PhysicalConstants() { }  // 인스턴스화 방지

  // 아보가드로 수 (1/몰)
  public static final double AVOGADROS_NUMBER = 6.022_140_857e23;

  // 볼츠만 상수 (J/K)
  public static final double BOLTZMANN_CONST  = 1.380_648_52e-23;

  // 전자 질량 (kg)
  public static final double ELECTRON_MASS    = 9.109_383_56e-31;
}
```
많이 사용된다면 정적 임포트를 고려하자.

2. 열거 타입으로 만들기
```java
public enum PhysicalConstants {
    AVOGADROS_NUMBER(6.022_140_857e23),
    BOLTZMANN_CONST(1.380_648_52e-23),
    ELECTRON_MASS(9.109_383_56e-31);

    private final double value;

    PhysicalConstants(double value) {
        this.value = value;
    }
}
```

#### 출처

이펙티브 자바 3/E

