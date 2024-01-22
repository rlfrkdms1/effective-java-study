# 열거타입

> **열거타입**은 일정 개수의 상수 값을 정의한 다음, 그 외의 값을 허용하지 않는 타입이다. 

## 정수 열거 패턴

```java
    public static final int APPLE_FUJI = 0;
    public static final int APPLE_PIPPIN = 1;
    public static final int APPLE_GRANNY_SMITH = 2;
    
    public static final int ORANGE_NAVEL = 0;
    public static final int ORANGE_TEMPLE = 1;
    public static final int ORANGE_BLOOD = 2;
```
위의 방식은 정수 열거 패턴 기법이다. 이 방법에는 단점이 많다.

### 단점

#### 스레드 안전 하지 않다.
언제든지 필드에 접근해 변경가능하기 때문이다. `final`이기 때문에 괜찮을 것 같지만, 가변 객체인 경우 `final`와 상관없이 변경될 여지가 있다. 

```java
package chap6;

import java.util.List;

public class IntEnumPattern {
   
    public static final int[] FRUIT = {0, 1, 2};

    public static void main(String[] args) {
        FRUIT[0] = 3;
        for (int i : FRUIT) {
            System.out.print(i + ", ");
        }
    }
}

```
위와 같이 배열의 각 원소에 접근 및 변경이 가능하다. 따라서 스레드안전하지 않다.

#### 타입 안전을 보장할 수 없다.

```java
package chap6;

import java.util.List;

public class IntEnumPattern {
    public static final int APPLE_FUJI = 0;
    public static final int APPLE_PIPPIN = 1;
    public static final int APPLE_GRANNY_SMITH = 2;

    public static final int ORANGE_NAVEL = 0;
    public static final int ORANGE_TEMPLE = 1;
    public static final int ORANGE_BLOOD = 2;

    public static void main(String[] args) {
        boolean grannySmith = isGrannySmith(APPLE_GRANNY_SMITH); // 올바른 예시
        boolean wrongGrannySmith = isGrannySmith(ORANGE_BLOOD); // 타입 안전 X
    }

    public static boolean isGrannySmith(int apple) {
        if (apple == 2) {
            return true;
        }
        return false;
    }
}
```
`apple`만을 매개변수로 받고 싶은 `isGrannySmith`라는 메서드가 있다고 할 때, `apple`만을 매개변수로 받는 것이 불가능하다. 따라서 언제나 `apple`일 것이라는 타입 안전을 보장할 수 없다. 

하지만 enum 타입을 사용하면 보장할 수 있다. 
```java
    public enum Apple {
        FUJI, PIPPIN, GRANNY_SMITH
    }
    
    public enum Orange {
        NAVEL, TEMPLE, BLOOD
    }


    public static void main(String[] args) {
        boolean grannySmith = isGrannySmith(Apple.GRANNY_SMITH);
        boolean wrongGrannySmith = isGrannySmith(Orange.NAVEL);
    }

    public static boolean isGrannySmith(Apple apple) {
        if (apple == Apple.GRANNY_SMITH) {
            return true;
        }
        return false;
    }
```

![](https://velog.velcdn.com/images/rlfrkdms1/post/fef66a8c-6f1b-4a2b-ac21-66f24d70ae37/image.png)

#### 표현력이 좋지 않다.

하나의 타입에 여러 속성이 있을 경우 극명히 드러난다. 예를 들어 `JIDAM`,`GAEUN`,`SUBIN`,`MINWOO`라는 이름을 가진 사람이 각각 여자2 남자2있고 각각 이름, 나이, 학년을 속성으로 가지고 있다. 그렇다면 아래와 같이 나타낼 수 있다. 

```java
    public static final String MAN_MINWOO_NAME = "MINWOO";
    public static final int MAN_MINWOO_AGE = 26;
    public static final int MAN_MINWOO_GRADE = 3;


    public static final String MAN_JIDAM_NAME = "JIDAM";
    public static final int MAN_JIDAM_AGE = 26;
    public static final int MAN_JIDAM_GRADE = 0;


    public static final String WOMAN_GAEUN_NAME = "GAEUN";
    public static final int WOMAN_GAEUN_AGE = 25;
    public static final int WOMAN_GAEUN_GRADE = 4;


    public static final String WOMAN_SUBIN_NAME = "SUBIN";
    public static final int WOMAN_SUBIN_AGE = 25;
    public static final int WOMAN_SUBIN_GRADE = 4;
```
굉장히 길고 가독성도 좋지 않다. 하지만 enum으로 나타내면 아래와 같다. 

```java
    public enum Man {
        MINWOO("Minwoo", 26, 3),
        JIDAM("Jidam", 26, 0);
        
        private final String name;
        private final int age;
        private final int grade;

        Man(String name, int age, int grade) {
            this.name = name;
            this.age = age;
            this.grade = grade;
        }
    }

    public enum Woman {
        GAEUN("Gaeun", 25, 4),
        SUBIN("Subin", 25, 4);

        private final String name;
        private final int age;
        private final int grade;

        Woman(String name, int age, int grade) {
            this.name = name;
            this.age = age;
            this.grade = grade;
        }
    }
```
각자 속성별로 정리되어 훨씬 읽고 분류하기 좋다. 

따라서 이런식으로 접두어를 붙여 상수를 분리하는 것은 좋지 않다. 또한, 이를 사용한 프로그램은 깨지기 쉽다. 컴파일시 그 값이 클라이언트 파일에 그대로 새려지는데, 상수의 값이 바뀌면 클라이언트도 반드시 다시 컴파일해야한다. 

또한 정수상수는 문자열로 출력하기가 다소 까다롭다. 이를 출력하거나 디버거로 보면 단지 숫자로만 보여 도움이 되지 않는다. 

Java에서는 열거 패턴의 단점을 말끔히 해결해주는 열거타입이라는 대안을 제시했다. 

## enum type

```java
public enum Human{
	GAEUN,SUBIN,MINWOO,JIDAM
}
```

아이디어는 단순하다. 열거타입 자체는 클래스이며, 상수 하나당 자신의 인스턴스를 하나씩 만들어 `public static final`필드로 공개한다.열거 타입은 밖에서 접근할 수 있는 생성자를 제공하지 않으므로 사실상 final 이다. 따라서 클라이언트가 인스턴스를 직접 생성하거나, 확장할 수 없어 인스턴스는 하나만 만들어진다.

위에서 우리는 정수 열거 타입의 단점을 enum 타입으로 극복하는 것을 보였다. 즉, enum type은 타입 안전하며,표현력이 좋다. 각자의 이름공간이 있어서 이름이 같은 상수도 평화롭게 공존한다. 

```java
public enum Human{
	GAEUN,SUBIN,MINWOO,JIDAM
}

public enum Woman{
	GAEUN,SUBIN
}
```

### 메서드, 필드 추가

열거타입에는 메서드와 필드를 추가할 수도, 임의의 인터페이스를 구현하게 할 수도 있다. 

```java
public enum Planet {
    MERCURY(3.302e+23, 2.439e6),
    VENUS  (4.869e+24, 6.052e6),
    EARTH  (5.975e+24, 6.378e6),
    MARS   (6.419e+23, 3.393e6),
    JUPITER(1.899e+27, 7.149e7),
    SATURN (5.685e+26, 6.027e7),
    URANUS (8.683e+25, 2.556e7),
    NEPTUNE(1.024e+26, 2.477e7);

    private final double mass;           // 질량(단위: 킬로그램)
    private final double radius;         // 반지름(단위: 미터)
    private final double surfaceGravity; // 표면중력(단위: m / s^2)

    // 중력상수(단위: m^3 / kg s^2)
    private static final double G = 6.67300E-11;

    // 생성자
    Planet(double mass, double radius) {
        this.mass = mass;
        this.radius = radius;
        surfaceGravity = G * mass / (radius * radius);
    }

    public double mass()           { return mass; }
    public double radius()         { return radius; }
    public double surfaceGravity() { return surfaceGravity; }

    public double surfaceWeight(double mass) {
        return mass * surfaceGravity;  // F = ma
    }
}
```
열거 타입 상수 각각을 특정 데이터와 연결지으려면 생성자에서 데이터를 받아 인스턴스 필드에 저장하면 된다. 

따라서 어떤 객체의 지구에서의 무게를 입력받아 여덟 행성에서의 무게를 출력하는 일을 다음처럼 짧은 코드로 작성이 가능하다. 

```java
public class WeightTable {
   public static void main(String[] args) {
      double earthWeight = Double.parseDouble(args[0]);
      double mass = earthWeight / Planet.EARTH.surfaceGravity();
      for (Planet p : Planet.values())
         System.out.printf("%s에서의 무게는 %f이다.%n",
                           p, p.surfaceWeight(mass));
   }
}
```
열거 타입은 자신 안에 정의된 상수들의 값을 순서대로 배열에 담아 반환하는 정적 메서드인 `values`를 제공한다.

또한 `toString()`을 활용해 상수 이름을 출력할 수도 있다.

열거 타입에서 상수를 하나 제거해도, 클라이언트에는 아무런 지장이 없다. `values`에 저장된 열거 타입 하나가 그냥 줄어드는 것 뿐이다. 

널리쓰이는 열거 타입은 톱레벨 클래스로 만들고, 특정 톱레벨 클래스에서만 쓰인다면 해당 클래스의 멤버 클래스로 만든다. 

### 상수마다 동작이 달라질 때

입력되는 연산 종류에 따라 연산을 해주고 싶을 때 우리는 switch를 이용한 동작을 생각할 수 있다. 

```java
package chap6;

public enum Operation {
    PLUS, MINUS, TIMES, DIVIDE;

    public double apply(double x, double y) {
        switch (this){
            case PLUS:
                return x + y;
            case MINUS:
                return x - y;
            case TIMES:
                return x * y;
            case DIVIDE:
                return x / y;
        }
        throw new AssertionError();
    }
}
```

하지만 이는 새로운 연산이 추가되면, case를 추가해야 하고 이를 까먹는다면, 새로운 연산을 수행할 때 런타임 에러가 날 것이다. 

따라서 추상메서드를 선언하고 각 상수별 클래스 몸체, 즉 각 상수에서 자신에 맞게 재정의하는 방법을 사용하면 더 좋다. 

```java
public enum Operation {
    PLUS("+") {
        public double apply(double x, double y) { return x + y; }
    },
    MINUS("-") {
        public double apply(double x, double y) { return x - y; }
    },
    TIMES("*") {
        public double apply(double x, double y) { return x * y; }
    },
    DIVIDE("/") {
        public double apply(double x, double y) { return x / y; }
    };

    private final String symbol;

    Operation(String symbol) { this.symbol = symbol; }

    public abstract double apply(double x, double y);
```

apply가 추상메서드이므로 이제 깜빡하고 재정의하지 않는다면, 컴파일 오류가 난다. 

### 상수별 메서드 구현을 상수별 데이터와 결합

toString 을 재정의해 연산을 뜻하는 기호를 반환하도록 해보자. 
```java
@Override public String toString() { return symbol; }
```

이를 사용하면 계산식 출력을 간편하게 사용할 수 있다. 

```java
    public static void main(String[] args) {
        double x = Double.parseDouble(args[0]);
        double y = Double.parseDouble(args[1]);
        for (Operation op : Operation.values())
            System.out.printf("%f %s %f = %f%n", x, op, y, op.apply(x, y));
    }
```
이를 실행하면 아래와 같은 결과가 나온다. 

```
2.000000 + 4.000000 = 6.000000
2.000000 - 4.000000 = -2.000000
2.000000 * 4.000000 = 8.000000
2.000000 / 4.000000 = 0.500000
```

`toString`을 재정의하려거든, `toString`이 반환하는 문자열을 해당 열거 타입 상수로 변환해주는 `fromString`메서드도 함께 제공하는 걸 고려해보자. 

```java
    private static final Map<String, Operation> stringToEnum =
            Stream.of(values()).collect(
                    toMap(Object::toString, e -> e));

    // 지정한 문자열에 해당하는 Operation을 (존재한다면) 반환한다.
    public static Optional<Operation> fromString(String symbol) {
        return Optional.ofNullable(stringToEnum.get(symbol));
    }
```
위 코드를 보면 연산 기호들을 key로 한 Map을 만들어 `fromString`을 통해 해당 연산 기호의 연산을 넘겨줬다. 이때 Optional 형태로 반환함으로써 클라이언트에게 주어진 문자열이 가리키는 연산이 존재하지 않을 수 있음을 알리고, 클라이언트가 대처하도록 한 것이다. 

### 상수별 메서드 구현에는 열거 타입 상수끼리 코드를 공유하기 어렵다.

```java
package chap6;

public enum PayrollDay {
    MONDAY, TUESDAY, WEDNESDAY, THURSDAY, FRIDAY, SATURDAY, SUNDAY;
    
    private static final int MINS_PER_SHIFT = 8 * 60;

    int pay(int minutesWorked, int payRate) {
        int basePay = minutesWorked * payRate;
        
        int overtimePay;
        switch (this) {
            case SATURDAY: case SUNDAY:
                overtimePay = basePay / 2;
                break;
            default:
                overtimePay = minutesWorked <= MINS_PER_SHIFT ? 0 : (minutesWorked - MINS_PER_SHIFT) * payRate / 2;
        }
        return basePay + overtimePay;
    }
}
```
위와 같이 기본 임금과, 일한 시간이 주어졌을 때 일당을 계산해주는 메서드를 갖고 있는 열거타입이 있다고 해보자. 주말에는 무조건 잔업 수당이 주어져 switch문으로 이를 해결할 수 있다. 위 코드는 간결해보이지만, 관리관점에서는 위험한 코드다. 휴가와 같은 새로운 값을 열거 타입에 추가하려면 그 값을 처리하는 case 문을 잊지 말고 넣어줘야한다.

```java
package chap6;

public enum PayrollDay {
    MONDAY, TUESDAY, WEDNESDAY, THURSDAY, FRIDAY, SATURDAY, SUNDAY, HOLIDAY,;

    private static final int MINS_PER_SHIFT = 8 * 60;

    int pay(int minutesWorked, int payRate) {
        int basePay = minutesWorked * payRate;

        int overtimePay;
        switch (this) {
            case SATURDAY: case SUNDAY: case HOLIDAY:
                overtimePay = basePay / 2;
                break;
            default:
                overtimePay = minutesWorked <= MINS_PER_SHIFT ? 0 : (minutesWorked - MINS_PER_SHIFT) * payRate / 2;
        }
        return basePay + overtimePay;
    }
}
```
case문을 깜빡한다면 예상과는 다른 급여를 받게 될 것이다. 

```java
package chap6;

public enum PayrollDay {
    MONDAY, TUESDAY, WEDNESDAY, THURSDAY, FRIDAY, SATURDAY, SUNDAY, HOLIDAY,;

    private static final int MINS_PER_SHIFT = 8 * 60;

    int pay(int minutesWorked, int payRate) {
        int basePay = minutesWorked * payRate;

        int overtimePay;
        switch (this) {
            case SATURDAY: case SUNDAY: 
                overtimePay = basePay / 2;
                break;
            default:
                overtimePay = minutesWorked <= MINS_PER_SHIFT ? 0 : (minutesWorked - MINS_PER_SHIFT) * payRate / 2;
        }
        return basePay + overtimePay;
    }
}
```
위와 같이 case문을 빼먹어도 잘 컴파일 되기 때문에 알아차리기 쉽지 않다. 그렇다면 이를 개선할 방법은 두가지다. 첫째, 잔업수당을 계산하는 코드를 모든 상수에 중복해서 넣으면 된다. 둘째, 계산 코드를 평일용과 주말용으로 나눠 각각을 도우미 메서드로 작성한 다음 각 상수가 자신에게 필요한 메서드를 적절히 호출하면 된다. 

하지만 위의 두 방법은 가독성이 크게 떨어지고 오류 발생 가능성이 높아진다.

따라서 잔업 수당 계산을 private 중첩 열거 타입으로 옮기고 PayrollDay 열거 타입의 생성자에서 이 중 적당한 것을 선택하면 된다. 아래의 코드를 보자. 

```java
enum PayrollDay {
    MONDAY(WEEKDAY), TUESDAY(WEEKDAY), WEDNESDAY(WEEKDAY),
    THURSDAY(WEEKDAY), FRIDAY(WEEKDAY),
    SATURDAY(WEEKEND), SUNDAY(WEEKEND);

    private final PayType payType;

    PayrollDay(PayType payType) { this.payType = payType; }
    // PayrollDay() { this(PayType.WEEKDAY); } // (역자 노트) 원서 4쇄부터 삭제
    
    int pay(int minutesWorked, int payRate) {
        return payType.pay(minutesWorked, payRate);
    }

    // 전략 열거 타입
    enum PayType {
        WEEKDAY {
            int overtimePay(int minsWorked, int payRate) {
                return minsWorked <= MINS_PER_SHIFT ? 0 :
                        (minsWorked - MINS_PER_SHIFT) * payRate / 2;
            }
        },
        WEEKEND {
            int overtimePay(int minsWorked, int payRate) {
                return minsWorked * payRate / 2;
            }
        };

        abstract int overtimePay(int mins, int payRate);
        private static final int MINS_PER_SHIFT = 8 * 60;

        int pay(int minsWorked, int payRate) {
            int basePay = minsWorked * payRate;
            return basePay + overtimePay(minsWorked, payRate);
        }
    }

    public static void main(String[] args) {
        for (PayrollDay day : values())
            System.out.printf("%-10s%d%n", day, day.pay(8 * 60, 1));
    }
}
```
PayrollDay 열거 타입은 잔업수당 계산을 전략 열거 타입에 위임해 switch문이나 상수별 메서드 구현이 필요없게 되었다. 따라서 이 패턴이 switch문 보다는 복잡하지만 더 안전하고 유연하다. 

하지만, 기존 열거 타입에 상수별 동작을 혼합해 넣을 때는 switch문이 좋은 선택이 될수 있다. 아래와 같은 상황일 때다. 

```java
public class Inverse {
    public static Operation inverse(Operation op) {
        switch(op) {
            case PLUS:   return Operation.MINUS;
            case MINUS:  return Operation.PLUS;
            case TIMES:  return Operation.DIVIDE;
            case DIVIDE: return Operation.TIMES;

            default:  throw new AssertionError("Unknown op: " + op);
        }
    }

    public static void main(String[] args) {
        double x = Double.parseDouble(args[0]);
        double y = Double.parseDouble(args[1]);
        for (Operation op : Operation.values()) {
            Operation invOp = inverse(op);
            System.out.printf("%f %s %f %s %f = %f%n",
                    x, op, y, invOp, y, invOp.apply(op.apply(x, y), y));
        }
    }
}
```
추가하려는 메서드가 의미상 열거 타입에 속하지 않는다면 직접 만든 열거 타입이라도 이 방식을 적용하는게 좋다. 

### 언제 쓸까?

#### 필요한 원소를 컴파일 타임에 알 수 있는 상수 집합이라면 항상 열거 타입을 사용하자. 

#### 열거 타입에 정의된 상수 개수가 영원히 고정 불변일 필요는 없다.


#### 출처

이펙티브 자바 3/E
[이펙티브 자바 github](https://github.com/WegraLee/effective-java-3e-source-code/tree/master)

