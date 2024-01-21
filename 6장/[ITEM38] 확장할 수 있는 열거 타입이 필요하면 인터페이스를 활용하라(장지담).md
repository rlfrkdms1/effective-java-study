```java
// 타입 안전 열거 패턴 
public class Animal {
    private String name;

    private Animal(String name) {
        this.name = name;
    }

    public static final Animal DOG = new Animal("dog");
    public static final Animal CAT = new Animal("cat");
}
```
- 타입 안전 열거 패턴은 jdk1.5 이전, enum을 지원하지 않을 때 사용하던 패턴이다. class이므로 확장할 수 있다. 
- enum은 java 문법 상 상속할 수 없다.

# 인터페이스 활용
```java
public interface Menu {
    public double vat(int percent);
}

public enum Pizza implements Menu {

    HAWAIIAN(10000) {
        @Override
        public double vat(int percent) {
            return HAWAIIAN.price * percent / 100;
        }
    },
    PEPERONI(12000) {
        @Override
        public double vat(int percent) {
            return PEPERONI.price * percent / 100;
        }
    };

    private final double price;

    private Pizza(double price) {
        this.price = price;
    }
}
```
- enum이 인터페이스를 구현하게 한다.
- 인터페이스의 메서드를 상수별로 구현한다. enum에 추상 메서드로 선언할 필요가 없다.


```java
public enum Chicken implements Menu {

    FRIED(17000) {
        @Override
        public double vat(int percent) {
            return FRIED.price * percent / 100;
        }
    },
    ROASTED(12000) {
        @Override
        public double vat(int percent) {
            return ROASTED.price * percent / 100;
        }
    };
    private final double price;

    private Chicken(double price) {
        this.price = price;
    }
}
```
- 클라이언트는 Menu 인터페이스를 구현해 자신만의 열거 타입 Chicken을 만들 수 있다. 
- 인터페이스 Menu를 사용하는 곳이라면 다형성에 의해 Menu를 확장한 enum Chicken을 사용할 수 있다.

# 확장한 enum의 원소 순회
## 방법 1

```java
public class Main {
    public static void main(String[] args) {
        showVats(Pizza.class,10);
        showVats(Chicken.class,10);
    }

    public static <T extends Enum<T> & Menu> void showVats(Class<T> menuEnumType, int percent) {
        for (Menu menu : menuEnumType.getEnumConstants()) {
            System.out.println(menu.vat(percent));
        }
    }
}
```
- `<T extends Enum<T> & Menu>`는 T가 enum 타입이면서 Menu를 구현해야 한다는 뜻이다.
  - enum 타입이어야 getEnumConstants로 원소를 순회할 수 있다
  - Menu의 구현체여야 Menu의 연산 vat을 사용할 수 있다 
- 한정적 타입 토큰 `Class<T> menuEnumType`를 매개변수로 받는다

## 방법2
```java
public class Main {
    public static void main(String[] args) {
        showVats(Arrays.asList(Pizza.values()), 10);
        showVats(Arrays.asList(Chicken.values()), 10);
    }

    public static void showVats(Collection<? extends Menu> menuSet, int percent) {
        for (Menu menu : menuSet) {
            System.out.println(menu.vat(percent));
        }
    }
}
```
- Menu의 구현체의 Collection을 받아 사용한다. 꼭 enum일 필요가 없어 유연하다.
- 대신 enum이어야만 할 수 있는 EnumSet, EnumMap들을 활용하지 못한다. 

# 열거 타입끼리 구현을 상속할 수 없다
- 새로운 enum Ramen을 만든다면 Chicken을 상속할 수 없다.

```java
public interface Menu {
    public default void cook() {
        System.out.println("요리 시작합니다.");
    }

    public double vat(int percent);
}
```
- 상태에 의존하지 않는다면 인터페이스에 default로 구현할 수 있다.
- 상태에 의존하는 중복되는 부분은 도우미 클래스나 정적 도우미 메서드로 분리할 수 있다.

