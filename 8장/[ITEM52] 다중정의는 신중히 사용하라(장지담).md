# 다중정의는 혼동을 일으킬 수 있다

```java
public class CollectionClassifier {
    public static String classify(Set<?> s) {
        return "집합";
    }

    public static String classify(List<?> lst) {
        return "리스트";
    }

    public static String classify(Collection<?> c) {
        return "그 외";
    }

    public static void main(String[] args) {
        Collection<?>[] collections = {
                new HashSet<String>(),
                new ArrayList<BigInteger>(),
                new HashMap<String, String>().values()
        };

        for (Collection<?> c : collections)
            System.out.println(classify(c));
    }
}
```
- 집합, 리스트, 그외 가 아닌 그 외만 3번 출력한다
- 이는 어느 메서드를 호출할 지는 컴파일타임에 정해지고 컴파일타임에는 인자가 모두 `Collection<?>` 타입이기 때문이다.
- 재정의(overiding) 한 메서드는 동적으로(실제 객체 타입에 맞게) 선택된다. 반면 다중정의(overloading)한 메서드는 정적으로(컴파일 타임 타입에 따라) 선택된다.
> 재정의 : 상위 클래스와 똑같은 시그니처의 메서드를 하위 클래스에서 정의한 것

```java
public class Robot {
    public void introduce() {
        System.out.println("i am robot");
    }
}

public class AutoBot extends Robot {
    @Override
    public void introduce() {
        System.out.println("i am auotobot");
    }
}

public class Client {
    public static void main(String[] args) {
        Robot r = new AutoBot();
        r.introduce();
    }
}
```
- 컴파일 타임에 r은 Robot 이지만 실제로는 AutoBot 타입이고 AutoBot 타입에서 재정의한 메서드가 호출되어 "i am autobot"을 출력한다.
- 반면 다중정의는 컴파일 타임의 타입에 따라 메서드가 호출된다.
- 프로그래머가 의도한 것은 재정의처럼 동작하는 것이다. 따라서 다중정의로 인해 혼란스러운 상황을 만들지 말아야 한다.
- 매개변수 수가 같은 다중정의는 만들지 말자.

# 다중정의 대신 메서드 이름을 다르게 지어주자
```java
    public void writeBoolean(boolean val) throws IOException {
        bout.writeBoolean(val);
    }

    public void writeByte(int val) throws IOException  {
        bout.writeByte(val);
    }

    public void writeShort(int val)  throws IOException {
        bout.writeShort(val);
    }

등등..
```
- ObjectOutputStream 클래스의 write 메서드는 `write자료형(자료형)`의 형태로 메서드를 선언했다.
- ObjectInputStream과 메서드 명을 맞추기도 용이하다. (`read자료형()`)

# 생성자
- 생성자는 이름을 다르게 지을 수 없어 2개 이상을 선언한다면 다중정의가 불가피하다. 다만 정적 팩터리를 사용할 수 있다.
- 생성자는 재정의 할 수 없어 재정의와 혼동할 일은 없다.
- 매개변수가 근본적으로 다르면 (두 타입의 값을 서로 형변환 할 수 없다면) 혼란이 사라진다.

# 다중정의 시 주의할 사항들 

### 제네릭과 오토박싱

```java
E remove(int index);
boolean remove(Object o);
```
- `List<E>`에서는 remove를 다중정의했다.
- 프로그래머는 오토박싱에 의해 `boolean remove(Object o)`를 호출할 것을 예상해도 `E remove(int index)`가 호출된다.
- 매개변수가 근본적으로 다를 것이라고 생각해도 제네릭과 오토박싱을 고려하자. 제네릭과 오토박싱에 의해 int가 Integer로 자동 형변환 된다.

### 람다와 메서드 참조
- 메서드를 다중정의할 때, 서로 다른 함수형 인터페이스라도 같은 위치의 인수로 받아서는 안된다. 서로 다른 함수형 인터페이스라도 근본적으로 다르지 않다.
- 다중정의 규칙은 매우 복잡하다.

> 매개변수 수가 같을 때는 다중정의를 피하는게 좋다.
