# 다중 구현 메커니즘

인터페이스와 추상클래스가 있다. 

## 공통점 

- 인스턴스 메서드를 구현형태로 제공

## 차이점

- 추상 클래스 : 추상 클래스가 정의한 타입을 구현하는 클래스는 반드시 추상 클래스의 하위 클래스가 되어야 함  -> 새로운 타입 정의에 제약
- 인터페이스 : 다른 어떤 클래스를 상속했든 같은 타입으로 취급

## 왜 인터페이스를 우선하는가?

### 상속
> 기존 클래스에도 손쉽게 새로운 인터페이스를 구현해 넣을 수 있다. 

두 클래스가 같은 추상 클래스 확장 -> 공통 조상 -> 계층 구조에 커다란 혼란

#### 추상 클래스
추상 클래스는 여러 클래스를 상속받지 못한다. 
```java
public abstract class A{
}

public abstract class B{
}

public class C extends A,B{
}
``` 
위처럼 extends A,B는 불가능하며 컴파일 에러가 뜬다. 따라서 하나만 상속 받을 수 있다. 

#### 인터페이스
```java
public interface class A{
}

public interface class B{
}

public class C implements A,B{
}
``` 
추상 클래스와 달리 여러개를 구현할 수 있다. 

### mixin
>인터페이스는 mixin 정의에 안성맞춤이다. 

mixin : 클래스가 구현할 수 있는 타입으로, 이를 구현한 클래스에 원래의 주된 타입 외에도 특정 선택적 행위를 제공한다고 선언하는 효과를 준다. ex) Comparable

클래스는 두 부모를 섬길 수 없고, 계층 구조상 믹스인을 삽입하기에 합리적인 위치가 없기 때문에 추상클래스로는 정의할 수 없다. 

```java
public class A implements Comparable,B {

  @Override
  public int compareTo(Object o) {
    return 0;
  }
}
```

### 타입 프레임워크
>인터페이스로는 계층구조가 없는 타입 프레임워크를 만들 수 잇다. 

```java
public interface Singer {
	AudioClip sing(Song s);
}

public interface Songwriter {
	Song compose(int chartPosition);
}
```
위와 같이 가수와 작곡가 인터페이스가 있다고 가정하자. 우리는 가수도 하면서, 직접 곡도 쓰는 사람을 정의하고 싶을 때 아래와 같이 정의할 수 있다. 

```java
public interface SingerSongwriter extends Singer, Songwriter{
}
```
 또한 기능도 추가할 수 있다. 
 
 ```java
public interface SingerSongwriter extends Singer, Songwriter{
	void acting();
}
```

위 구조를 클래스로 만드려면 가능한 조합들을 모두 클래스로 정의한 커다란 계층구조가 만들어질 것이다. 

속성이 n개라면 지원해야할 조합의 수는 2^n개나 된다. (조합 폭발 현상)
```java
public abstract class Singer {
	AudioClip sing(Song s);
}

public abstract class Songwriter {
	Song compose(int chartPosition);
}

public abstract class SingerSongwriter{
	AudioClip sing(Song s);
	Song compose(int chartPosition);
	void acting();
}

public class Person extends SingerSongwriter{
}
```


### default 메서드의 제약
- equals, hashCode같은 Object의 메서드는 default 메서드로 정의 X
- 인터페이스는 인스턴스 필드를 가질 수 없고 public이 아닌 정적 멤버도 가질 수 없다.(private 정적 메서드는 예외)
- 내가 만들지 않은 인터페이스에는 디폴트 메서드를 추가할 수 없다. 

## 추상 골격 구현 클래스

인터페이스로는 타입을 정의하고, 골격 구현 클래스는 나머지 메서드들까지 구현 => 골격 구현을 확장하는 것만으로 인터페이스를 구현하는데 필요한 일이 거의 완료 => "템플릿 메서드 패턴"

관례상 인터페이스 이름이 Interface라면 골격 구현 클래스 이름은 AbstractInterface로 짓는다. (예. AbstractCollection)

```java
public class IntArrays {
    static List<Integer> intArrayAsList(int[] a) {
        Objects.requireNonNull(a);

        return new AbstractList<>() {
            @Override public Integer get(int i) {
                return a[i];  // 오토박싱(아이템 6)
            }

            @Override public Integer set(int i, Integer val) {
                int oldVal = a[i];
                a[i] = val;     // 오토언박싱
                return oldVal;  // 오토박싱
            }

            @Override public int size() {
                return a.length;
            }
        };
    }

    public static void main(String[] args) {
        int[] a = new int[10];
        for (int i = 0; i < a.length; i++)
            a[i] = i;

        List<Integer> list = intArrayAsList(a);
        Collections.shuffle(list);
        System.out.println(list);
    }
}
```
- 추상클래스처럼 구현을 도와줌
- 추상 클래스로 타입을 정의할 때 따라오는 심각한 제약에서는 자유로움


구조상 골격 구현을 확장하지 못한다면 인터페이스를 직접 구현해야함 
인터페이스를 구현한 클래스에서 해당 골격 구현을 확장한 private 내부 클래스를 정의하고 각 메서드 호출을 내부 클래스의 인스턴스에 전달하면 된다. 

#### 골격 구현 작성 방법
1. 인터페이스에서 다른 메서드들의 구현에 사용되는 기반 메서드들 선정
2. 기반 메서드를 사용해 직접 구현할 수 있는 메서드를 모두 디폴트 메서드로 제공
     - Object의 메서드는 제공 X
3. 기반 메서드나, 디폴트 메서드로 만들지 못한 메서드가 남아 있다면 골격 구현 클래스를 하나 만들어 남은 메서드들을 작성
    - 만약 남은 메서드가 없다면 골격 구현 클래스를 만들 필요 X
```java
public abstract class AbstractMapEntry<K,V>
        implements Map.Entry<K,V> {
    // 변경 가능한 엔트리는 이 메서드를 반드시 재정의해야 한다.
    @Override public V setValue(V value) {
        throw new UnsupportedOperationException();
    }
    
    // Map.Entry.equals의 일반 규약을 구현한다.
    @Override public boolean equals(Object o) {
        if (o == this)
            return true;
        if (!(o instanceof Map.Entry))
            return false;
        Map.Entry<?,?> e = (Map.Entry) o;
        return Objects.equals(e.getKey(),   getKey())
                && Objects.equals(e.getValue(), getValue());
    }

    // Map.Entry.hashCode의 일반 규약을 구현한다.
    @Override public int hashCode() {
        return Objects.hashCode(getKey())
                ^ Objects.hashCode(getValue());
    }

    @Override public String toString() {
        return getKey() + "=" + getValue();
    }
}
```

조금 더 간단한 예제를 보자. 아래는 골격화를 사용하기 전이다. 
```java
public interface Student {

    void eat();

    void study();

    void goHome();

    void dailyRoutine();
}

public class Gil implements Student {

    @Override
    public void study() {
        System.out.println("클래스와 인터페이스 공부를 해요.");
    }
    
    @Override
    public void eat() {
        System.out.println("학식을 먹어요");
    }

    @Override
    public void goHome() {
        System.out.println("집으로 돌아가요");
    }

    @Override
    public void dailyRoutine() {
        eat();
        study();
        goHome();
    }
}

public class Jang implements Student {

    @Override
    public void study() {
        System.out.println("객체의 생성과 파괴를 공부해요");
    }
    @Override
    public void eat() {
        System.out.println("학식을 먹어요");
    }

    @Override
    public void goHome() {
        System.out.println("집으로 돌아가요");
    }

    @Override
    public void dailyRoutine() {
        eat();
        study();
        goHome();
    }
}
```
그럼 골격화를 사용하면 어떻게 될까? 
```java

public interface Student {

    void eat();

    void study();

    void goHome();

    void dailyRoutine();
}


public abstract class DguStudent implements Student {

    @Override
    public void eat() {
        System.out.println("학식을 먹어요");
    }

    @Override
    public void goHome() {
        System.out.println("집으로 돌아가요");
    }

    @Override
    public void dailyRoutine() {
        eat();
        study();
        goHome();
    }
}

public class Jang extends DguStudent implements Student {
    @Override
    public void study() {
        System.out.println("객체의 생성과 파괴를 공부해요");
    }
}

public class Gil extends DguStudent implements Student {

    @Override
    public void study() {
        System.out.println("클래스와 인터페이스 공부를 해요.");
    }
}

public class Main {

    public static void main(String[] args) {

        List<Student> students = new ArrayList<>();
        Student jang = new Jang();
        Student gil = new Gil();
        students.add(jang);
        students.add(gil);

        for (Student student : students) {
            student.dailyRoutine();
        }
    }
}

```
```
학식을 먹어요
객체의 생성과 파괴를 공부해요
집으로 돌아가요
학식을 먹어요
클래스와 인터페이스 공부를 해요.
집으로 돌아가요
```
중복되는 코드도 줄일 수 있고, 한층 더 유연성을 갖게 되었다. 

#### 출처

이펙티브 자바 3/E
