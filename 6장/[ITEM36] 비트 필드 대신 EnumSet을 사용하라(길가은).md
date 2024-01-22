열거한 값들이 주로 집합으로 사용될 경우, 예전에는 각 상수에 서로 다른 2의 거듭제곱 값을 할당한 정수 열거 패턴을 사용해왔다.

```java
public class Text {
    public static final int STYLE_BOLD = 1 << 0; //1
    public static final int STYLE_ITALIC = 1 << 1; //2
    public static final int STYLE_UNDERLINE = 1 << 2; //4
    public static final int STYLE_STRIKETHROUGH = 1 << 3; //8
   
   //STYLE_상수 OR 연산
    public void applyStyles(int styles) {
        ...
    }
}

```

다음과 같은 식으로 비트별 OR을 사용해 여러 상수를 하나의 집합으로 모을 수 있으며, 이렇게 만들어진 집합을 비트 필드라 한다. 

```java
text.applyStyles(STYLE_BOLD | STYLE_ITALIC);
```

비트 필드를 사용하면, 집합 연산을 효율적으로 수행할 수 있지만 아래의 문제점이 있다. 

1. 비트 필드 값이 그대로 출력되면 단순한 정수 열거 상수를 출력할 때보다 해석하기 훨씬 어렵다. 

2. 비트 필드 하나에 녹아 있는 모든 원소를 순회하기 까다롭다. 

3. 최대 몇 비트가 필요한지를 API 작성 시 미리 예측해 적절한 타입 (int or long)을 선택해야한다.

따라서 대안으로 EnumSet을 사용할 수 있다. 아래와 같이 수정할 수 있다. 

```java
public class Text {
    public enum Style {BOLD, ITALIC, UNDERLINE, STRIKETHROUGH}

    // 어떤 Set을 넘겨도 되나, EnumSet이 가장 좋다.
    public void applyStyles(Set<Style> styles) {
        System.out.printf("Applying styles %s to text%n",
                Objects.requireNonNull(styles));
    }

    // 사용 예
    public static void main(String[] args) {
        Text text = new Text();
        text.applyStyles(EnumSet.of(Style.BOLD, Style.ITALIC));
    }
}
```

장점
1. Set 인터페이스를 구현해 타입 안전하고 다른 어떤 Set 구현체와도 함께 사용할 수 있다. 

2. 내부가 비트 벡터로 구현되어 대부분의 경우 EnumSet 전체를 long 변수 하나로 표현해 비트 필드에 비견되는 성능을 보여준다. 

3. EnumSet이 난해한 작업을 모두 처리하므로 비트를 직접 다룰때의 오류들에서 해방된다.



#### 출처

이펙티브 자바 3/E
[이펙티브 자바 github](https://github.com/WegraLee/effective-java-3e-source-code/tree/master)
