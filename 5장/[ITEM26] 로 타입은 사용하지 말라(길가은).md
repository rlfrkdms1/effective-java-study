> 클래스와 인터페이스 선언에 타입 매개변수가 쓰이면, 이를 **제네릭 클래스** 혹은 **제네릭 인터페이스**라 한다. 

제네릭 클래스와 제네릭 인터페이스를 통틀어 제네릭 타입이라 한다. 

각각의 제네릭 타입은 일련의 매개변수화 타입을 정의한다.  `List<String>`은서 원소타입이 `String`인 리스트를 뜻하는 매개변수화 타입이다. 여기서 `String`이 정규 타입 매개변수 E에 해당하는 실제 타입 매개변수다. 

# raw type
> 제네릭 타입에서 타입 매개변수를 전혀 사용하지 않을 때를 말한다. 

ex. `List<E>` 의 raw type : List

타입 선언에서 제네릭 타입 정보가 전부 지워진 것 처럼 동작 -> 제네릭이 도래하기 전 코드와 호환하기 위함

## 사용하면 안되는 이유
```java
private final Collection stamps = ...;
```
위에서는 로 타입인 `Collection`을 사용하고 있는 것을 볼 수 있다. 이를 사용하면 의도는 Stamp의 인스턴스만을 취급하는 것이지만, `Collection`에 Stamp대신 Coin을 넣어도 아무 오류 없이 컴파일 되어 실행된다. 

```java
stamps.add(new Coin(..));
```
위와 같이 넣으면 "unchecked call" 경고를 내뱉을 뿐이다. 

```java
import java.util.ArrayList;
import java.util.List;

public class Main {

    public static void main(String[] args) {
        List stamps = new ArrayList();
        stamps.add(new Stamp());
        stamps.add(new Coin());
    }
}
```
즉, 위와 같은 일이 가능하다는 것이다. 하지만, 아래와 같은 상황이 된다고 해보자. 

```java
package item26;

import java.util.ArrayList;
import java.util.List;

public class Main {

    public static void main(String[] args) {
        List stamps = new ArrayList();
        stamps.add(new Stamp());
        stamps.add(new Coin());

        for (Object item : stamps) {
            Stamp stamp = (Stamp) item;
            System.out.println("done!");
        }
    }
}
```
그렇다면? `ClassCastException`가 터지게 되는 것이다. 즉, 우리가 예상치 못했던 Coin이 아무런 주의없이 stamps에 들어갈 수 있게 되었고 이는 런타임 에러로 발견되었다. 만약 stamps를 `add()`를 통해 생성하는 부분과 stamps를 Stamp로 캐스팅하는 부분이 떨어져있다면, 굉장히 찾기 어려운 버그가 될 것이다. 

![](https://velog.velcdn.com/images/rlfrkdms1/post/5ae544fb-82cd-4da2-9b3b-679787dc0ceb/image.png)

따라서 아래와 같이 선언해서 사용하자. 

```java
import java.util.ArrayList;
import java.util.List;

public class Main {

    public static void main(String[] args) {
        List<Stamp> stamps = new ArrayList<>();
        stamps.add(new Stamp());
        stamps.add(new Coin());

        for (Object item : stamps) {
            Stamp stamp = (Stamp) item;
            System.out.println("done!");
        }
    }
}
```
그렇다면 아래와 같이 컴파일에러가 나서 금방 잡을 수 있는 버그가 되며, Stamp를 넣는 List 라는 것이 명시되어 있으므로, 다른 프로그래머가 실수로 Coin을 넣는 일도 없을 것이다. 

```
'add(item26.Stamp)' in 'java.util.List' cannot be applied to '(item26.Coin)'
```

> 로 타입을 쓰면 제네릭이 안겨주는 안전성과 표현력을 모두 잃게 된다. 

그렇다면 왜 쓰는걸까? 앞서 말했듯 호환성 때문이다. 

`List`는 사용해선 안된다고 했지만 `List<Object>`는 괜찮다. 모든 타입을 허용한다는 의사를 컴파일러에게 명확히 전달했기 때문이다. 

매개변수로 `List`를 받는 메서드에 `List<String>`을 넘길 수 있지만, `List<Object>`를 받는 메서드에는 넘길 수 없다. 이는 제네릭의 하위 타입 규칙때문이다. 

> 즉, `List<String>`은 로 타입인 `List`의 하위 타입이지만, `List<Object>`의 하위타입은 아니다. 

그 결과 `List<Object>`같은 매개변수와 타입을 사용할 때와 달리 List같은 로타입을 사용하면 타입 안전성을 잃게 된다. 

```java
package item26;

import java.util.ArrayList;
import java.util.List;

public class Main {

    public static void main(String[] args) {
        List<String> stringList = new ArrayList<>();
        rawType(stringList, Integer.valueOf(1));
    }

    private static void rawType(List list, Object o) {
        list.add(o);
        System.out.println("o in list's class = " + list.get(0).getClass().getSimpleName() + " o's class = " + o.getClass().getSimpleName());
    }
}
```
위의 코드를 실행하면 아래와 같다. 

```
o in list's class = Integer o's class = Integer
```
어라 우리는 `List<String>`에 Integer 원소를 넣었는데, list에 들어있는게 Integer라고 한다. 그렇다면 stringList에서 원소를 꺼내보자. 

```java
stringList.get(0);
```
을 해주면 아래와 같은 런타임 에러가 뜬다. 
```
Exception in thread "main" java.lang.ClassCastException: class java.lang.Integer cannot be cast to class java.lang.String (java.lang.Integer and java.lang.String are in module java.base of loader 'bootstrap')
	at item26.Main.main(Main.java:11)
```
그렇다면 로타입을 `List<Object>`로 바꿔보자. 

```java
import java.util.ArrayList;
import java.util.List;

public class Main {

    public static void main(String[] args) {
        List<String> stringList = new ArrayList<>();
        rawType(stringList, Integer.valueOf(1));
        String item = stringList.get(0);
        System.out.println("item.getClass().getSimpleName() = " + item.getClass().getSimpleName());
    }

    private static void rawType(List<Object> list, Object o) {
        list.add(o);
        System.out.println("o in list's class = " + list.get(0).getClass().getSimpleName() + " o's class = " + o.getClass().getSimpleName());
    }
}
```
다음과 같은 컴파일에러가 뜨는 것을 알 수 있다. 

```
'rawType(java.util.List<java.lang.Object>, java.lang.Object)' in 'item26.Main' cannot be applied to '(java.util.List<java.lang.String>, java.lang.Integer)'
```

`List<String>`은 `List<Object>`의 하위 타입이 아니기 때문이다.

## 비한정적 와일드 카드

예제를 하나 더 살펴보자.

```java
    static int numElementsInCommon(Set s1, Set s2) {
        int result = 0;
        for (Object o : s1) {
            if (s2.contains(o)) {
                result++;
            }
        }
        return result;
    }
```
두 Set에 공통적으로 속해있는 원소의 개수를 조회하는 메서드인데, 로타입을 매개변수로 받았다. 이 메서드는 동작하지만, 로타입을 사용해 안전하지 않다. 
이때 사용할 수 있는 것이 비한정적 와일드 카드 타입이다. 

아래와 같은 상황이라고 생각해보자. 

```java
        Collection<?> wild = new ArrayList<>();
        Collection raw = new ArrayList<>();
        wild.add("item");
        raw.add("item");
```
비한정적 와일드 카드 타입을 사용하면, null 외의 원소는 추가할 수 없다. 따라서 위에서는 `wild.add("item");`에서 컴파일 에러가 발생한다. 

즉, 로 타입에서는 아무 원소나 추가가 가능해 불변식을 훼손할 수 있지만 비한정적 와일드 카드 타입을 사용하면 null이외의 원소를 넣지 못해 안전하다. 따라서 방금과 같이 두 Set 사이의 공통원소의 개수를 조회하는 메서드에서는 원소 추가가 되면 안되므로 비한정적 와일드 카드 타입을 매개변수로 사용해야한다. 

```java
   static int numElementsInCommonRaw(Set s1, Set s2) {
        int result = 0;
        for (Object o : s1) {
            if (s2.contains(o)) {
                result++;
                s1.add("hi");
            }
        }
        return result;
    }
```
위 코드는 raw type으로 중간에 s1.add("hi");를 해도 에러가 뜨지 않고 동작되어 훼손 될 수 있다. 

하지만 아래의 비한정적 와일드 카드 타입을 사용한 메서드는 s1.add("hi")에서 컴파일 에러가 떠 안전하다. 

```java
    static int numElementsInCommonWild(Set<?> s1, Set<?> s2) {
        int result = 0;
        for (Object o : s1) {
            if (s2.contains(o)) {
                result++;
                s1.add("result"); //컴파일 에러
                s1.add(null); // 괜찮음
            }
        }
        return result;
    }
```

## raw type을 써야할 때 

### class 리터럴

자바 명세는 class 리터럴에 매개변수화 타입을 사용하지 못하게 했다. (배열과 기본 타입은 허용)

ex. `List.class`, `String[].class`, `int.class` : O
	`List<String>`, `List<?>.class` : X
    
## 제네릭 타입에 instanceof 사용

```java
        if (o instanceof Set) {
            Set<?> s = (Set<?>) o;
            ...
        }
```
비한정적 와일드 타입이과 로 타입이 똑같이 동작하므로 `<?>`를 사용해 지저분하게 하는 것 보단 로 타입을 사용하는 것이 낫다. 


#### 출처 

이펙티브 자바 3/E
