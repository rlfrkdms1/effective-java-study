선택적 매개변수가 많을 때 적용할 수 있는 3가지 패턴이 있다. 

# 점층적 생성자 패턴 

```java
    public Robot(int battery, int power) {
        this.battery = battery;
        this.power = power;
    }

    public Robot(int battery, int power, int age) {
        this.battery = battery;
        this.power = power;
        this.age = age;
    }

    public Robot(int battery, int power, int age, int fuel) {
        this.battery = battery;
        this.power = power;
        this.age = age;
        this.fuel = fuel;
    }
```

점층적 생성자 패턴은 필수 매개변수만 받는 생성자, 필수 매개변수 + 선택 매개변수 1개 받는 생성자, 필수 매개변수 + 선택 매개변수 2개 받는 생성자.. 의 방식으로 생성자를 선언하는 패턴이다.
- 사용자가 원치 않는 매개변수까지 포함하는 경우가 많고, 원치 않는 매개변수까지 값을 지정해줘야 한다.
- 클라이언트가 실수로 매개변수 순서를 바꿔 건내줘도 컴파일타임에 잡지 못한다.

# 자바빈즈 패턴
```java
        Robot robot = new Robot(1, 2);
        robot.setAge(3);
        robot.setFuel(10);
```
자바빈즈 패턴은 setter들로 값을 설정하는 방식이다. 점층적 생성자 패턴에 비해 인스턴스를 만들기 쉽고, 읽기 쉽다.
- 객체 하나를 만들려면 메서드를 여러개 호출해야 한다.
- 객체가 완전히 생성되기 전까지는 일관성이 무너진 상태에 놓인다.
- 매개변수가 유효한지 생성자에서 확인할 수 없다.
- 불변 클래스로 만들 수 없다.

# 빌더 패턴 
점층적 생성자 패턴의 안전성 + 자바빈즈 패턴의 가독성 
```java
public class Robot {
    private final int power;
    private final int age;
    private final int velocity;
    private final int battery;

    public static class Builder {
        // 필수매개변수
        private final int power;
        private final int age;

        // 선택매개변수
        private int velocity = 10;
        private int battery = 0;

        // 빌더 생성자
        public Builder(int power, int age) {
            if (age < 1) {
                throw new IllegalArgumentException();
            }
            this.power = power;
            this.age = age;
        }

        // 빌더 세터, 빌더 자기 자신을 반환함
        public Builder velocity(int val) {
            if (velocity < 1) {
                throw new IllegalArgumentException();
            }
            velocity = val;
            return this;
        }

        public Builder battery(int val) {
            if (battery < 1) {
                throw new IllegalArgumentException();
            }
            battery = val;
            return this;
        }

        public Robot build() {
            return new Robot(this);
        }
    }

    private Robot(Builder builder) {
        power = builder.power;
        age = builder.age;
        velocity = builder.velocity;
        battery = builder.battery;
    }
}
```
- 필수 매개변수만으로 생성자 or 정적 팩터리 메서드를 호출해서 빌더 객체를 얻는다.
- 빌더 객체를 이용해 선택 매개변수들을 설정한다. 
- build() 를 호출해 객체를 얻는다.
- 빌더의 생성자와 세터에서 검증한다.
- build()에서 호출하는 외부 클래스 생성자에서 불변식을 검사한다.
- 불변 객체를 생성할 수 있다.
- 외부 객체 생성자는 private이며, 해당 객체의 Builder로만 생성할 수 있다.
  
Builder는 클래스 안에 정적 멤버 클래스로 만든다. 빌더의 세터는 빌더 자기 자신을 반환해서 연쇄적으로 호출이 가능하므로 flunet api 또는 method chaining이라 한다.
```java
Robot robot = new Robot.Builder(1,2)
                .velocity(2)
                .build();
```
클라이언트는 빌더의 생성자로 빌더를 생성하고 세터로 선택매개변수 값을 지정하고 `build()`를 호출해서 외부 객체 Robot을 얻는다. 
- 원하는 선택적 파라미터에만 값을 지정할 수 있으며, 필드 이름과 함께 지정하므로 명시적이다.

## 클래스 계층구조에서 빌더 패턴
```java
public abstract class Player {
    private final int speed;

    public abstract static class Builder<T extends Builder<T>> {
        private int speed;

        public T speed(int speed) {
            if (speed < 1) {
                throw new IllegalArgumentException();
            }
            this.speed = speed;
            return self();
        }

        abstract Player build();

        protected abstract T self();
    }

    Player(Builder builder) {
        speed = builder.speed;
    }
}
```
- 추상 클래스에는 추상 빌더를 정의한다. 재귀적 타입 한정을 사용한다. 이것은 구체 클래스 빌더의 세터의 반환 타입을 자기 자신으로 강제하기 위함이다. 
- 모든 서브타입에서 공통적으로 사용할 필드를 선언한다.
- 이 필드의 setter를 선언한다. (빌더 방식으로)
- 자기 자신을 반환하는 추상 메서드 self를 선언한다.

```java
public class SoccerPlayer extends Player {
    private final int shoot;

    public static class Builder extends Player.Builder<Builder> {

        private final int shoot;

        public Builder(int shoot) {
            if (shoot < 1) {
                throw new IllegalArgumentException();
            }
            this.shoot = shoot;
        }

        @Override
        SoccerPlayer build() {
            return new SoccerPlayer(this);
        }

        @Override
        protected Builder self() {
            return this;
        }
    }

    private SoccerPlayer(Builder builder) {
        super(builder);
        shoot = builder.shoot;
    }
}
```
- 추상 빌더 Player.Builder를 구체 빌더 Builder가 상속한다. 타입매개변수 T는 Player.Builder를 상속하는 Builder다. 따라서 self는 Builder를 반환한다.
- 구체 클래스에서는 구체 빌더를 구현한다.
- 오버라이딩 한 메서드들의 반환 타입은 구체 타입이다. 즉 하위 클래스 메서드가 상위 클래스에서 정의한 반환형이 아닌 그 반환형의 하위 타입을 반환하는 공변 반환 타이핑이다.
- 공변 반환 타이핑을 사용하면 클라이언트는 형변환을 신경쓰지 않고 빌더를 사용할 수 있다.

```java
        SoccerPlayer soccerPlayer = new Builder(20)
                .speed(10)
                .build();
```
- 추상 클래스 Player의 필드 speed를 설정할 수 있다. `speed()`의 반환타입은 `self()`에 의해 구체 클래스 빌더인 SoccerPlayer.Builder이다.

  ---

**빌더 패턴의 단점**
- 빌더 생성 비용이 발생한다
- 매개변수가 4개 이상은 되어야 의미있다. 하지만 api는 매개변수가 적었다 많아지는 경향이 있다. 따라서 애초에 빌더를 선택하는 것이 좋을 때가 많다. 

  
