함수 객체를 만드는 주요 수단은 익명 클래스이다. 

```java
        Collections.sort(words, new Comparator<String>() {
            @Override
            public int compare(String o1, String o2) {
                return Integer.compare(o1.length(), o2.length());
            }
        });
```
예전에는 위의 기법처럼 익명 클래스를 구현해 사용했으나, 코드가 너무 길어 함수형 프로그래밍에 적합하지 않았다. 

따라서 추상 메서드 하나짜리 인터페이스는 함수형 인터페이스가 되어 람다식을 사용해 만들 수 있도록 되었다. 람다는 함수나 익명 클래스와 개념은 비슷하지만 코드는 훨씬 간결하다. 위의 코드는 어떻게 바뀌었을까?

```java
Collections.sort(words, (s1, s2) -> Integer.compare(s1.length(), s2.length()));
```

한줄로 바뀐 것을 알 수 있다. 람다, 매개변수 (s1, s2)와 반환값의 타입은 각각 (`Comparator<String>`, String, int)이지만 람다식을 사용한 위의 코드에서는 보이지 않는다. 우리 대신 컴파일러가 문맥을 살펴 타입을 추론한 것이다. 

만약 컴파일러가 타입을 결정하지 못한다면, 프로그래머가 직접 명시해야한다. 

>타입을 명시해야 코드가 더 명확할 때만 제외하고는, 람다의 모든 매개변수 타입은 생략하자. 

람다 자리에 비교자 생성 메서드를 사용하면 코드를 더 간결하게 만들 수 있다. 

```java
Collections.sort(words, comparingInt(String::length));
```
List인터페이스의 sort 메서드를 사용하면 더 짧게 사용가능하다. 

```java
words.sort(comparingInt(String::length));
```

item 34에서 enum의 사용법에 대해 다뤘었다. 그때 정의했던 `Operation` 클래스가 기억나는가? 아래와 같다. 

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

    @Override public String toString() { return symbol; }

    public abstract double apply(double x, double y);
```

item 34에서는 상수별 클래스 몸체를 구현하는 것보다 열거 타입에 인스턴스 필드를 두는 편이 낫다고 했다. 람다를 이용하면 이것이 가능하다. 

```java
public enum Operation {
    PLUS  ("+", (x, y) -> x + y),
    MINUS ("-", (x, y) -> x - y),
    TIMES ("*", (x, y) -> x * y),
    DIVIDE("/", (x, y) -> x / y);

    private final String symbol;
    private final DoubleBinaryOperator op;

    Operation(String symbol, DoubleBinaryOperator op) {
        this.symbol = symbol;
        this.op = op;
    }

    @Override public String toString() { return symbol; }

    public double apply(double x, double y) {
        return op.applyAsDouble(x, y);
    }
```
각 열거 타입 상수의 동작을 람다로 구현해 생성자에 넘기고 생성자는 이 람다를 인스턴스 필드로 저장해둔다. 이후 `apply`메서드에서 이 람다를 호출하기만 하면 된다. 

이를 보면 상수별 클래스 몸체는 더이상 사용할 이유가 없다고 생각할 수 있지만, 그렇지 않다. 

> 람다는 이름이 없고 문서화도 못한다. 따라서 코드 자체로 동작이 명확히 설명되지 않거나 코드 줄 수가 많아지면 람다를 쓰지 말아야한다. 

람다는 한 줄일 때 가장 좋고 길어야 세 줄 안에 끝내는게 좋다. 세 줄을 넘어가면 가독성이 심하게 나빠진다. 따라서 람다가 길거나 읽기 어렵다면 더 간단히 줄여보거나 람다를 쓰지 않는 쪽으로 리팩터링해야한다. 

열거 타입 생성자에 넘겨지는 인수들의 타입도 컴파일 타임에 추론되기때문에 인스턴스 필드인 람다는 열거 타입의 인스턴스 멤버에 접근할 수 없다. 따라서 인스턴스 필드나, 메서드를 사용해야하는 상황이면 상수별 클래스 몸체를 사용해야한다. 

그렇다면, 람다로 대체할 수 없는 것은 무엇이있을까?

람다는 함수형 인터페이스에서만 쓰인다. 

- 추상 클래스의 인스턴스를 만들 때 람다를 쓸 수 없으니, 익명 클래스를 써야한다. 

```java
public abstract class Human {
    protected String name;

    public abstract void eat(String food);

    public abstract void sleep(int time);

    public void hi() {
        System.out.println("Nice to meet you");
    }
}
```

```java
public class Example {
    public static void main(String[] args) {
        Human human = new Human() {
            @Override
            public void eat(String food) {
                System.out.printf("%s 식사", food);
            }

            @Override
            public void sleep(int time) {
                System.out.printf("%시간 취침", time);
            }

            @Override
            public void hi() {
                super.hi();
            }
        };

        human.eat("라면");
    }
    
}
```

- 추상 메서드가 여러 개인 인터페이스의 인스턴스를 만들 때, 익명 클래스를 사용할 수 있다.

- 람다는 자신을 참조할 수 없어 람다에서 `this`는 바깥 인스턴스를 가리키지만 익명 클래스에서 `this`는 자신을 가리킨다. 

> 람다도 익명 클래스처럼 직렬화 형태가 구현별로 다를 수 있어 람다를 직렬화하는 일은 극히 삼가야 한다. 

#### 출처

이펙티브 자바 3/E


[이펙티브 자바 github](https://github.com/WegraLee/effective-java-3e-source-code/tree/master)
