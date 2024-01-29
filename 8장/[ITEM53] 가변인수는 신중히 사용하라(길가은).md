가변인수 메서드는 명시한 타입의 인수를 0개 이상 받을 수 있다. 즉, 인수가 없어도 동작한다. 

가변인수는 아래의 순서로 동작한다. 

1. 인수의 개수와 길이가 같은 배열을 만든다. 
2. 인수들을 만든 배열에 저장한다. 
3. 배열을 가변인수 메서드에 건네준다. 

다음의 예제를 보자. 

```java

public class Calculation {

    static int sum(int... args) {
        int sum = 0;
        for (int arg : args) {
            sum += arg;
        }
        return sum;
    }

}

public class Main {

    public static void main(String[] args) {

        System.out.println(Calculation.sum(1, 2, 3));
        System.out.println(Calculation.sum());

    }
}
```
출력 결과는 아래와 같다. 

```
6
0
```

그렇다면 매개변수가 꼭 1개 이상이어야 하는 가변인수 메서드를 만들고 싶다면 어떨까?

다음과 같이 생각할 수 있다. 

```java
    static int min(int... args) {
        if (args.length == 0) {
            throw new IllegalArgumentException("가변인수가 1개 이상 필요합니다.");
        }
        int min = args[0];
        for (int arg : args) {
            if (arg < min) {
                min = arg;
            }
        }
        return min;
    }
```
이 메서드에는 두가지 문제가 있다. 

1. 인수를 0개 넣으면 컴파일 타임이 아닌 런타임에 예외를 던지며 프로그램이 실패하는 것이다. 
2. `args`의 유효성 검사를 명시적으로 해야해 코드가 지저분하다. 

이는 가변인수의 길이를 런타임에 알 수 있기에 생긴 문제다. 따라서 최소한의 인수는 가변인수로 받지 않도록 해 문제를 해결할 수 있다. 

```java
    static int min(int firstArg, int... remainingArgs) {
        int min = firstArg;
        for (int arg : remainingArgs) {
            if (arg < min) {
                min = arg;
            }
        }
        return min;
    }
```    
인수로 들어올 매개변수 중 첫 번째 매개변수를 가변인수가 아닌 매개변수로 받고, 나머지 매개변수들을 가변인수로 받으면 문제가 없다. firstArg에 매개변수를 채워넣지 않으면 컴파일타임 에러가 뜨기 때문이다. 

가변인수는 인수 개수가 정해지지 않았을 때 아주 유용하다. `printf`도 이러한 가변인수의 장점을 활용한 것이다. 

```java
    public PrintStream printf(String format, Object ... args) {
        return format(format, args);
    }
```

하지만 성능이 중요한 상황에서는 메서드가 호출될 때 마다 배열을 할당하고 초기화하는 가변인수 메서드가 안좋을 수 있다. 

따라서 이러한 상황(가변인수의 유연성은 필요하나 성능에 민감)에 사용할 수 있는 패턴이 있다. 

예를 들어 해당 메서드 호출의 95%가 인수를 3개 이하로 가진다고 해보자. 그렇다면 아래와 같이 재정의를 활용해 문제를 해결할 수 있다. 

```java
    public void execute() {}
    public void execute(int a) {}
    public void execute(int a, int b) {}
    public void execute(int a, int b, int c) {}
    public void execute(int a, int b, int c, int... args) {}
```
위와 같이 한다면 메서드 호출 중 95%를 가변인수를 활용하지 않는 메서드를 사용하게 될 것이다. 그렇다면 5%의 호출에서만 가변인수를 사용하므로 성능면에 나은 것이다. 

이 기법도 보통때는 별 이득이 없으나, 꼭 필요한 특수 상황에서는 성능 개선에 도움을 줄 것이다. 

EnumSet도 이 기법을 사용한다.
```java
    public static <E extends Enum<E>> EnumSet<E> of(E e) {
        EnumSet<E> result = noneOf(e.getDeclaringClass());
        result.add(e);
        return result;
    }
 
    public static <E extends Enum<E>> EnumSet<E> of(E e1, E e2) {
        EnumSet<E> result = noneOf(e1.getDeclaringClass());
        result.add(e1);
        result.add(e2);
        return result;
    }

    public static <E extends Enum<E>> EnumSet<E> of(E e1, E e2, E e3) {
        EnumSet<E> result = noneOf(e1.getDeclaringClass());
        result.add(e1);
        result.add(e2);
        result.add(e3);
        return result;
    }

    public static <E extends Enum<E>> EnumSet<E> of(E e1, E e2, E e3, E e4) {
        EnumSet<E> result = noneOf(e1.getDeclaringClass());
        result.add(e1);
        result.add(e2);
        result.add(e3);
        result.add(e4);
        return result;
    }

    public static <E extends Enum<E>> EnumSet<E> of(E e1, E e2, E e3, E e4, E e5)
    {
        EnumSet<E> result = noneOf(e1.getDeclaringClass());
        result.add(e1);
        result.add(e2);
        result.add(e3);
        result.add(e4);
        result.add(e5);
        return result;
    }

    @SafeVarargs
    public static <E extends Enum<E>> EnumSet<E> of(E first, E... rest) {
        EnumSet<E> result = noneOf(first.getDeclaringClass());
        result.add(first);
        for (E e : rest)
            result.add(e);
        return result;
    }
```

#### 출처

이펙티브 자바 3/E


