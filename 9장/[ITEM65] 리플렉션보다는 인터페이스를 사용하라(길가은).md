리플렉션 기능을 사용하면 프로그램에서 임의의 클래스에 접근할 수 있다. Class 객체가 주어졌을 때 그 클래스의 생성자, 메서드, 필드에 해당하는 인스턴스들을 가져올 수 있고 이 인스턴스로 부터 클래스의 멤버 이름, 필드 타입, 메서드 시그니처등을 가져올 수 있다. 

```java
package chap9.item65;

public class Dog {
    private String name;
    private int age;

    public Dog(String name) {
        this.name = name;
    }

    public Dog(String name, int age) {
        this.name = name;
        this.age = age;
    }

    public void eat() {
        System.out.println("밥을 먹어요");
    }

    public void sound() {
        System.out.println("멍멍");
    }
}
```
위와 같은 클래스가 있다고 가정할 때 리플렉션을 사용하면 아래와 같은 정보를 얻을 수 있다. 

```java
public class Main {
    public static void main(String[] args) {
        Class clazz = Dog.class;
        Constructor[] constructors = clazz.getConstructors();
        for (Constructor constructor : constructors) {
            System.out.println("constructor = " + constructor);
        }
        Method[] methods = clazz.getMethods();
        for (Method method : methods) {
            System.out.println("method = " + method);
        }
        Field[] fields = clazz.getFields();
        for (Field field : fields) {
            System.out.println("field = " + field);
        }

    }
}
```

출력결과는 아래와 같다. 
```
constructor = public chap9.item65.Dog(java.lang.String,int)
constructor = public chap9.item65.Dog(java.lang.String)
method = public void chap9.item65.Dog.eat()
method = public void chap9.item65.Dog.sound()
method = public final native void java.lang.Object.wait(long) throws java.lang.InterruptedException
method = public final void java.lang.Object.wait(long,int) throws java.lang.InterruptedException
method = public final void java.lang.Object.wait() throws java.lang.InterruptedException
method = public boolean java.lang.Object.equals(java.lang.Object)
method = public java.lang.String java.lang.Object.toString()
method = public native int java.lang.Object.hashCode()
method = public final native java.lang.Class java.lang.Object.getClass()
method = public final native void java.lang.Object.notify()
method = public final native void java.lang.Object.notifyAll()
```
또한 아래와 같이 메서드를 알고 있을 때 클래스를 알 수 있다. 
```java
        Class clazz = Dog.class;
        Method eat = clazz.getMethod("eat");
        System.out.println(eat.getDeclaringClass());
```

```
class chap9.item65.Dog
```

또한 Constructor, Mehod, Field 인스턴스를 통해 각각에 연결된 실제 생성자, 메서드, 필드를 조작할 수도 있다. 

Dog 클래스의 필드인 name을 public static으로 바꾸고 아래와 같이 "mong"이라는 이름을 줬다고 하자. 

```java
public class Dog {
    public static String name = "mong";
    ...
}
```
그렇다면 리플렉션을 통해 name을 바꾸는 것이 가능하다. 

```java
        Class clazz = Dog.class;
        Field name = clazz.getField("name");
        name.set(null, "hi");
        System.out.println(name.get(null));
```
출력결과는 아래와 같다. 
```
hi
```
필드가 "mong"에서 "hi"로 바뀐 것을 볼 수 있다. 

즉, 이를 통해 인스턴스들을 통해 해당 클래스의 인스턴스를 생성하거나, 메서드를 호출하거나, 필드에 접근할 수 있다. 

예로 `Metho.d.invoke`를 사용하면 어떤 클래스의 어떤 객체가 가진 어떤 메서드라도 호출할 수 있게 해준다. 또한 리플렉션을 이용하면 컴파일 당시에 존재하지 않던 클래스도 이용할 수 있는데, 물론 단점이 있다. 


- 컴파일타임 타입 검사가 주는 이점을 하나도 누릴 수 없다. 
	- 예외 검사도 마찬가지다. 
    - 런타임 오류가 발생할 확률이 늘어난다. 
- 리플렉션을 이용하면 코드가 지저분하고 장황해진다. 
- 성능이 떨어진다. 
	- 리플렉션을 이용한 메서드 호출은 일반 호출보다 훨씬 느리다. 

> 리플렉션은 아주 제한된 형태로만 사용해야 그 단점을 피하고 이점만 취할 수 있다. 

컴파일타임에 이용할 수 없는 클래스를 사용해야만 하는 프로그램은 비록 컴파일 타임이라도 적절한 인터페이스나 상위클래스를 이용할 수는 있을 것이다. 

> 리플렉션은 인스턴스 생성에만 쓰고, 이렇게 만든 인스턴스는 인터페이스나 상위 클래스로 참조해 사용하자. 

```java
public class ReflectiveInstantiation {

    public static void main(String[] args) {
        // 클래스 이름을 Class 객체로 변환
        Class<? extends Set<String>> cl = null;
        try {
            cl = (Class<? extends Set<String>>)  // 비검사 형변환!
                    Class.forName(args[0]);
        } catch (ClassNotFoundException e) {
            fatalError("클래스를 찾을 수 없습니다.");
        }

        // 생성자를 얻는다.
        Constructor<? extends Set<String>> cons = null;
        try {
            cons = cl.getDeclaredConstructor();
        } catch (NoSuchMethodException e) {
            fatalError("매개변수 없는 생성자를 찾을 수 없습니다.");
        }

        // 집합의 인스턴스를 만든다.
        Set<String> s = null;
        try {
            s = cons.newInstance();
        } catch (IllegalAccessException e) {
            fatalError("생성자에 접근할 수 없습니다.");
        } catch (InstantiationException e) {
            fatalError("클래스를 인스턴스화할 수 없습니다.");
        } catch (InvocationTargetException e) {
            fatalError("생성자가 예외를 던졌습니다: " + e.getCause());
        } catch (ClassCastException e) {
            fatalError("Set을 구현하지 않은 클래스입니다.");
        }

        // 생성한 집합을 사용한다.
        s.addAll(Arrays.asList(args).subList(1, args.length));
        System.out.println(s);
    }

    private static void fatalError(String msg) {
        System.err.println(msg);
        System.exit(1);
    }
}
```
위의 프로그램은 인수로 받은 Set 클래스의 인스턴스를 생성해 사용한다. 

두가지 단점을 보여준다. 

- 런타임에 총 여섯가지나 되는 예외를 던질 수 있다. 
	- 리플렉션 없이 생성했다면 컴파일타임에 잡아낼 수 있었을 예외다. 
- 클래스 이름만으로 인스턴스를 생성해내기 위해 25줄이나 되는 코드를 작성했다. 
	- 리플렉션이 아니었다면 생성자 호출이면 되었다. 
    
    
하지만 이 단점들은 인스턴스를 생성할때만으로 국한되고 생성된 이후는 리플렉션을 사용하지 않을 때와 동일하다.     

리플렉션은 런타임에 존재하지 않을 수도 있는 다른 클래스, 메서드, 필드와의 의존성을 관리할 때 적합하다. 이 기법은 버전이 여러개 존재하는 외부 패키지를 다룰 때 유용하다. 

가동할 수 있는 최소한의 환경, 즉 주로 가장 오래된 버전만을 지원하도록 컴파일한 후 이후 버전의 클래스와 메서드등은 리플렉션으로 접근하는 방식이다. -> 런타임에 존재하지 않을 수 있음을 꼭 명심 !

#### 출처

이펙티브 자바 3/E
