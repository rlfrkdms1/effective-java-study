# equals를 재정의하지 않아야 할 상황
equals를 재정의하지 않아도 되는 상황에는 재정의하지 않는 것이 best다. 
- 각 인스턴스가 본질적으로 고유하다
  - 값이 아닌 동작하는 개체 표현, Thread
- 논리적 동치성을 검사할 일이 없다 
- 상위 클래스 equals가 하위 클래스에 들어맞는다
  - 대부분의 Set 구현체는 AbstractSet의 equals를 상속받아 쓴다.
- 클래스가 private이거나 package-priavte이고 equals를 호출할 일이 없다.
  - equals가 실수로라도 호출되는걸 막고 싶다면 오버라이딩해서 예외를 던지도록 하자
```java
    @Override
    public boolean equals(Object obj) {
        throw new AssertionError();
    }
```

# equals를 재정의해야 할 상황
**논리적 동치성을 확인해야 하는데 상위 클래스의 equals가 논리적 동치성을 비교하도록 재정의되지 않았을 때**
- 주로 값 클래스
  - 객체가 같은지(객체 식별성)가 아닌 값이 같은지(논리적 동치성)이 궁금하다
  - 값 클래스더라도 값이 같은 객체가 2개 이상 만들어지지 않음이 보장되는 인스턴스 통제 클래스라면 equals를 재정의하지 않아도 된다.(Enum)
- 논리적 동치성을 비교하도록 재정의하먼 Map, Set등의 원소로 사용할 수 있다

# equals 일반 규약
equals를 재정의 할 때는 반드시 일반 규약을 따라야 한다.
```java
public class Robot {
    private String name;
    private Integer age;

    public Robot(String name, Integer age) {
        this.name = name;
        this.age = age;
    }
}
```
Robot 클래스를 활용해 하나씩 살펴보자.
## 반사성 
- **null이 아닌  모든 참조 값 x에 대해 x.equals(x)는 true다.**
- 객체는 자기 자신과 같아야 한다. 
- Collection의 contains로 확인할 수 있다.
```java
    @Override
    public boolean equals(Object obj) {
        if (obj == null) {
            return false;
        }
        return this == obj;
    }
```
```java
    public static void main(String[] args) {
        Robot r1 = new Robot("mong",10);
        List<Robot> list = new ArrayList<>();
        list.add(r1);
        System.out.println(list.contains(r1)); // true 
    }
```
## 대칭성 
- **null이 아닌  모든 참조 값 x, y에 대해 x.equals(y)가 true면 y,equlas(x)도 true다.**
- 서로에 대한 동치 여부에 똑같이 답해야 한다.
```java
    @Override
    public boolean equals(Object obj) {
        if (obj == null) {
            return false;
        }
        if (obj instanceof Robot) {
            return name.equals(((Robot) obj).name);
        }
        if (obj instanceof String) {
            return name.equals((String) obj);
        }
        return false;
    }
```
Robot의 name 필드만 갖고 동치 여부를 판단하는 equals를 생각해보자.
```java
    public static void main(String[] args) {
        Robot r1 = new Robot("mong",10);
        Robot r2 = new Robot("mong",20);
        String name = "mong";
        System.out.println(r1.equals(r2)); // true 
        System.out.println(r1.equals(name)); // true
        System.out.println(name.equals(r1)); // false
    }
```
String의 equals는 Robot 객체를 모르기 때문에 세번째 경우에 대해 false를 얻고, 반사성을 위반한다.
```java
    public static void main(String[] args) {
        Robot r1 = new Robot("mong",10);
        Robot r2 = new Robot("mong",20);
        String name = "mong";

        List<Robot> list = new ArrayList<>();
        list.add(r1);
        System.out.println(list.contains(r2)); // true
        System.out.println(list.contains(name)); // false
    }
```
```java
    int indexOfRange(Object o, int start, int end) {
        Object[] es = elementData;
        if (o == null) {
            for (int i = start; i < end; i++) {
                if (es[i] == null) {
                    return i;
                }
            }
        } else {
            for (int i = start; i < end; i++) {
                if (o.equals(es[i])) {
                    return i;
                }
            }
        }
        return -1;
    }
```
- ArrayList의 contains는 indexOfRange 반환값이 0 이상이면 true를 반환한다
- indexOfRange에서 인자로 들어온 Object의 equals를 호출해서 동치를 판단한다. 즉 `list.contains(name)`에서는 String의 equals가 호출되므로 contains의 결과가 false인 것이다.
  - indexOfRange의 구현이 `es[i].equals(o)`로 변경된다면 true를 반환할 것이다. 
- 대칭성을 만족하지 않으면 객체를 사용하는 다른 객체들의 동작을 예측할 수 없다.
```java
    @Override
    public boolean equals(Object obj) {
        if (obj == null) {
            return false;
        }
        if (obj instanceof Robot) {
            return name.equals(((Robot) obj).name);
        }
        return false;
    }
```
대칭성을 만족하기 위해서는 Robot의 equals를 String과 연동되게 하겠다는 꿈을 버려야 한다.
## 추이성 
- **null이 아닌  모든 참조 값 x, y, z에 대해 x.equals(y)가 true고 y,equlas(z)가 true면 x.equals(z)도 true다.**
```java
    @Override
    public boolean equals(Object obj) {
        if (!(obj instanceof Robot)) {
            return false;
        }
        Robot robot = (Robot) obj;
        return name.equals(robot.name) && age.equals(robot.age);
    }
```
```java
public class DogRobot extends Robot {
    private Integer bark;

    @Override
    public boolean equals(Object obj) {
        if (!(obj instanceof DogRobot)) {
            return false;
        }
        return super.equals(obj) && ((DogRobot) obj).bark.equals(bark);
    }
}
```
- DogRobot에서 equals를 오버라이딩 하지 않으면 DogRobot의 정보인 bark를 비교할 수 없다.
- equals의 대상이 또다른 DogRobot이고 이름, 나이, 짖기가 모두 같아야 true를 반환하게 재정의했다. 
```java
    public static void main(String[] args) {
        Robot r1 = new Robot("mong", 10);
        DogRobot dr = new DogRobot("mong", 10, 100);

        System.out.println(r1.equals(dr)); // true
        System.out.println(dr.equals(r1)); // false
    }
```
- Robot의 equals와 DogRobot의 equals가 다르게 동작한다. 대칭성을 위반한다.
```java
    @Override
    public boolean equals(Object obj) {
        if (!(obj instanceof Robot)) {
            return false;
        }
        //obj가 Robot일 경우 bark는 비교하지 않는다
        if (!(obj instanceof DogRobot)) {
            return obj.equals(this);
        }
        return super.equals(obj) && ((DogRobot) obj).bark.equals(bark);
    }
```
- 비교 대상 객체가 Robot일 경우는 bark를 비교하지 않고, DogRobot일 경우는 bark까지 비교하도록 equals를 재정의했다.
```java
    public static void main(String[] args) {
        DogRobot dr1 = new DogRobot("mong", 10, 100);
        Robot r1 = new Robot("mong", 10);
        DogRobot dr2 = new DogRobot("mong", 10, 200);

        System.out.println(dr1.equals(r1)); // true
        System.out.println(r1.equals(dr2)); // true
        System.out.println(dr1.equals(dr2)); // false
    }
```
- 대칭성은 만족하지만, 추이성을 위반한다.
- 무한 재귀 문제에 빠질 수 있다. equals를 똑같이 구현한 CatRobot 구현체를 만들고 `catRobot.equals(dogRobot)`을 하면 서로의 equals를 무한히 호출하게 된다.
- **구체 클래스를 확장해 새로운 값을 추가하면서 equals 규약을 만족시킬 방법은 없다**
```java
    @Override
    public boolean equals(Object obj) {
        if (obj == null || obj.getClass() != getClass()) {
            return false;
        }
        DogRobot o = (DogRobot) obj;
        return super.equals(o) && bark.equals(o.bark);
    }
```
- instanceof 대신 getClass를 사용하면 규약도 지키고, 값도 추가할 수 있다.
- 하지만 리스코프 치환 원칙(LSP)을 위배한다. 같은 타입끼리 비교할 때만 true를 반환한다.
> LSP : 어떤 타입에 있어 중요한 속성이라면 하위 타입에서도 중요하다. 그 타입의 모든 메서드가 하위 타입에서도 똑같이 잘 작동해야 한다.  
- DogRobot도 Robot이므로 Robot으로 활용될 수 있어야 한다.
- 예외적으로 추상 클래스에서는 equals 규약을 지키면서 구체 클래스에 값을 추가할 수 있다. 추상 클래스 객체는 생성할 수 없기 때문이다.

```java
public class DogRobot {
    private final Robot robot;
    private final Integer bark;

    public DogRobot(String name, Integer age, Integer bark) {
        robot = new Robot(name, age);
        this.bark = bark;
    }

    // DogRobot의 Robot 뷰를 반환한다
    public Robot asRobot() {
        return robot;
    }

    @Override
    public boolean equals(Object obj) {
        if (!(obj instanceof DogRobot)) {
            return false;
        }
        DogRobot o = (DogRobot) obj;
        return o.robot.equals(robot) && o.bark.equals(bark);
    }
}
```
- 구체 클래스의 하위 클래스에서 값을 추가하는 것에 대한 우회 방법으로 상속 대신 컴포지션을 사용할 수 있다(ITEM18)
- Robot을 상속하는 대신 DogRobot의 private 필드로 선언한다.
- Robot을 반환하는 뷰 메서드를 public으로 추가한다.

## 일관성
- **null이 아닌  모든 참조 값 x, y에 대해 x.equals(y)를 반복해서 호출하면 항상 true거나 항상 false다.**
- 두 객체가 같다면, 수정되지 않는 한 영원히 같아야 한다
- 불변 객체인 경우 한번 같으면 영원히 같고 한번 다르면 영원히 다르다.
- equals 계산에 신뢰할 수 없는 자원이 끼어들어서는 안된다. 항시 메모리에 존재하는 결정적 계산만 수행해야 한다.

```java
    public boolean equals(Object obj) {
        if (!(obj instanceof URL u2))
            return false;

        return handler.equals(this, u2);
    }
```
```java
    protected boolean sameFile(URL u1, URL u2) {
        // Compare the protocols.
        if (!((u1.getProtocol() == u2.getProtocol()) ||
              (u1.getProtocol() != null &&
               u1.getProtocol().equalsIgnoreCase(u2.getProtocol()))))
            return false;

        // Compare the files.
        if (!(u1.getFile() == u2.getFile() ||
              (u1.getFile() != null && u1.getFile().equals(u2.getFile()))))
            return false;

        // Compare the ports.
        int port1, port2;
        port1 = (u1.getPort() != -1) ? u1.getPort() : u1.handler.getDefaultPort();
        port2 = (u2.getPort() != -1) ? u2.getPort() : u2.handler.getDefaultPort();
        if (port1 != port2)
            return false;

        // Compare the hosts.
        if (!hostsEqual(u1, u2))
            return false;

        return true;
    }
```
- URL의 equals는 handler.equals의 값을 반환한다
- handler.equals는 비교 과정에서 sameFile을 호출하고 sameFile에서는 hostEqual을 호출한다.
- hostEqual은 네트워크를 통해 호스트를 IP로 바꿔서 비교하고, 결과가 항상 같다고 보장할 수 없다.
- URL의 equals는 일관성을 어긴 메서드다.

## null 아님
- **null이 아닌  모든 참조 값 x에 대해 x.equals(null)은 false다.**
- 모든 객체는 null과 같지 않아야 한다
- NPE를 던지는 것이 아니라 false를 반환해야 한다.
- instanceof 연산자는 첫번째 피연산자가 null이면 false를 반환하는 묵시적 null 체크를 한다. 따라서 명시적 null 체크가 필요없다.

# 올바른 equals 구현방법
1. `==` 연산자로 입력이 자기 자신의 참조인지 확인한다. 자기 자신이면 true를 반환한다.
   - 이것은 비교 작업이 복잡할 때 성능을 위해서이다.
2. instanceof 연산자로 입력이 올바른 타입인지 확인한다. 그렇지 않다면 false를 반환한다.
   - 올바른 타입은 일반적으로 equals가 구현된 클래스를 의미한다
   - 올바른 타입은 가끔은 그 클래스가 구현한 특정 인터페이스가 될 수도 있다.
     - 어떤 인터페이스는 자신을 구현한 서로 다른 클래스들끼리도 비교할 수 있게 equals를 수정하기도 한다. 이러한 인터페이스의 구현클래스에서 equals를 재정의 할 때는 instanceof에서 해당 인터페이스를 사용해야 한다.
     - Set,List,Map,Map.entry등이 이에 해당한다.
   - 묵시적 null 체크를 효과도 있다.
3. 입력을 올바른 타입으로 형변환한다.
   - 앞의 단계에서 instanceof로 검사했기 때문에 무조건 성공한다.
4. 입력 객체와 자기 자신의 대응되는 핵심 필드들이 모두 일치하는지 하나씩 검사한다. 모두 일치하면 true, 아니면 false를 반환한다
    - 2단계에서 인터페이스를 사용했다면 입력의 필드값을 가져올 때도 인터페이스의 메서드를 사용해야 한다.
    - float, double을 제외한 기본 타입 필드는 ==로 검사한다.
    - float, double은 각각 Float.compare(float, float)과 Dobule.compare(double, double)로 비교한다. 
    - 배열의 모든 원소가 핵심 필드라면 Arrays.equals 메서드들 중 하나를 사용하자
    - null도 정상 값으로 취급하는 필드라면 Objects.equals(Object, Object)로 비교해 NPE를 예방하자
    - 비교하기 복잡한 필드를 가진 클래스는 표준형을 저장해놓고 표준형끼리 비교하면 경제적이다.
    - 필드를 비교하는 순서가 성능에 영향을 미치기도 한다
      - 다를 가능성이 크거나 비교 비용이 저렴한 필드를 먼저 비교하자
      - 객체의 논리적 상태와 관련 없는 필드는 비교하지 않는다.
      - 핵심 필드로 계산할 수 있는 파생 필드는 비교할 필요가 없지만 파생 필드가 객체 전체의 상태를 대변한다면 파생 필드를 비교하는 게 나을 수도 있다.
```java
public class Robot {
    private String name;
    private int age;

    public Robot(String name, int age) {
        this.name = name;
        this.age = age;
    }

    @Override
    public boolean equals(Object obj) {
        if (obj == this) {
            return true;
        }
        if (!(obj instanceof Robot)) {
            return false;
        }
        Robot o = (Robot) obj;
        return name.equals(o.name) && age == o.age;
    }
}
```
---

- equals를 다 구현했다면 대칭적인가?추이성이 있는가?일관적인가?를 자문하자.
- 단위 테스트를 작성해 돌려보자.
- 구글의 AutoValue 프레임워크를 활용하면 클래스에 애너테이션만 붙이면 테스트코드를 알아서 작성해준다

# equals 재정의 시 주의점
- equals를 재정의 할 때는 hashCode도 반드시 재정의해야 한다. 
- 너무 복잡하게 해결하려 들지 말자. 
- 입력 타입은 반드시 Object여야 한다. 입력 타입이 Object가 아닐 경우 오버라이딩이 아니라 오버로딩 한 것이다.


