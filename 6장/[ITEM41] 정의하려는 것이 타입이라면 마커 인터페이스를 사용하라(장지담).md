# 마커 인터페이스
- 아무런 메서드도 없고, 자신을 구현하는 클래스가 특정 속성을 가짐을 표시해주는 인터페이스
- Serlializable, Cloneable 등 
```java
public interface Serializable {
}
```
- Serializable은 자신을 구현한 클래스의 인스턴스는 직렬화 할 수 있음을 표시해준다.

# 마커 애노테이션 
- 아무 매개변수 없이 대상에 마킹하는 애너테이션
- 프로그램 요소가 특정 속성을 가짐을 나타낸다 
- @Overide

# 마커 인터페이스의 장점
마커 인터페이스는 마커 애노테이션에 비해 2가지 장점을 갖는다. 
1. 마커 인터페이스는 이를 구현한 클래스의 인스턴스들을 구분하는 타입으로 쓸 수 있다.
```java
    public static void main(String[] args) {
        run(new Robot());
    }
    
    public static void run(Runnable runnable){
        
    }
```
- 마커 인터페이스 Runnable은 타입으로 사용할 수 있다. 
- Robot 클래스는 Runnable을 구현하지 않아서 컴파일에러가 발생한다.

  ![image](https://github.com/rlfrkdms1/effective-java-study/assets/96513365/369ffa5c-31b4-40e1-ae54-8b5f2fce0e59)

- 애노테이션은 타입으로 사용할 수 없다. 런타임에 문제를 발견한다.
2. 적용 대상을 더 정밀하게 지정할 수 있다.
- 적용 대상 @Target을 Element.TYPE으로 선언한 애너테이션은 모든 타입, 즉 클래스, 인터페이스, 열거타입, 애너테이션 에 적용할 수 있다. 더 세밀하게 제한할 수는 없다.
```java
public interface Runnable {
}

public interface Robotics extends Runnable{
}
```
- 특정 인터페이스를 구현한 클래스에만 적용하고 싶은 마커 인터페이스가 있다면 마커 인터페이스가 특정 인터페이스를 상속하게 한다.
- 마커 인터페이스 Robotics가 마킹된 대상은 자동으로 Runnable을 구현해야 한다.

# 마커 애너테이션의 장점 
- 거대한 애너테이션 시스템의 지원을 받는다
- 애너테이션을 적극 활용하는 프레임워크에서는 애너테이션을 사용하는 것이 통일성 측면에서 낫다.

---

클래스나 인터페이스에 마커를 달 때, 해당 클래스나 인터페이스를 매개변수로 받는 메서드를 작성할 일이 있다면 마커 인터페이스를 사용하자. 컴파일타임에 에러를 잡아낼 수 있다. 

