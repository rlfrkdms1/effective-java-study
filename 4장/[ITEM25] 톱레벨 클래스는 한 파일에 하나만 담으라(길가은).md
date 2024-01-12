```java
public class Main {
    public static void main(String[] args) {
        System.out.println(Utensil.NAME + Dessert.NAME);
    }
}

class Utensil {
    static final String NAME = "pan";
}

class Dessert {
    static final String NAME = "cake";
}
``` 
위와 같이 되어있다고 하자. 이때 Utensil, Dessert클래스는 모두 Utensil.java에 존재한다. main을 실행하면 "pancake"가 잘 출력된다. 그런데 이때 아래와 같은 Dessert.java가 생겼다고 해보자. 

```java
class Utensil {
    static final String NAME = "pot";
}

class Dessert {
    static final String NAME = "pie";
}
```

이는 컴파일 에러를 일으킨다. Main이 컴파일 되어 Utensil 참조를 만나면 Utensil.java를 통해 Utensil와 Dessert가 있음을 알게될 것이다. 그리고 두번째 인자인 Dessert.NAME을 만나 Dessert.java를 방문하면 같은 클래스의 정의가 있음을 깨닫는다. 

javac를 사용해 컴파일 해보자. 

- pancake 출력
  ```
  javac Main.java
  javac Main.java Utensil.java
  ```
- potpie 출력
  ```
  javac Dessert.java Main.java
  ```
위에서 볼 수 있읏 컴파일러가 건네는 파일의 순서에 따라 실행되는 것이 다르다. 
이 문제의 해결방법은 단순히 톱레벨 클래스들을 서로 다른 소스 파일로 분리하면 그만이다. 

굳이 여러 톱레벨 클래스를 한 파일에 담고 싶다면 정적 멤버 클래스를 사용하는 방법을 고민해 볼 수 있다. 읽기 좋고, private으로 선언하면 접근 범위도 최소로 관리할 수 있기 때문이다. 

```java
public class Test {
    public static void main(String[] args) {
        System.out.println(Utensil.NAME + Dessert.NAME);
    }

    private static class Utensil {
        static final String NAME = "pan";
    }

    private static class Dessert {
        static final String NAME = "cake";
    }
}
```
