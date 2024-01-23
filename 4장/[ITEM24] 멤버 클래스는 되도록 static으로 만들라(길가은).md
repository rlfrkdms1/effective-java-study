중첩 클래스는 자신을 감싼 바깥 클래스에서만 쓰여야하며 그 외의 쓰임새라면 톱 레벨 클래스로 만들어야 한다. 종류는 정적 멤버 클래스, 멤버 클래스, 익명 클래스, 지역 클래스 네가지 이다. 정적 멤버 클래스외에는 내부 클래스에 해당한다. 

# 중첩 클래스 언제, 왜 사용해요?
## 정적 멤버 클래스
바깥 클래스의 private 멤버에도 접근할 수 있고 다른 클래스 안에서 선언된다는 점 빼고는 일반 클래스와 같다. 정적 멤버 클래스는 다른 정적 멤버와 똑같은 접근 규칙을 적용받는다. private으로 선언하면 바깥 클래스에서만 접근할 수 있다. 

```java
public class Example {

    private int value;
    public static class PublicEx{
        public void access() {
            Example example = new Example();
            example.value = 10;

            PrivateEx privateEx = new PrivateEx();
            privateEx.access();
            System.out.println("PublicEx.access");
        }
    }

    private static class PrivateEx{
        public void access() {
            Example example = new Example();
            example.value = 5;
            System.out.println("PrivateEx.access");
        }
    }
}

```
바깥 클래스의 private 멤버에도 접근할 수 있음을 알 수 있다.
![](https://velog.velcdn.com/images/rlfrkdms1/post/01fcba70-4a6b-42ff-aad7-e1be8fcda15e/image.png)
하지만 이를 외부에서 사용하려하면 private 으로 선언된 클래스엔 접근이 되지 않음을 알 수 있다. 

중첩 클래스의 인스턴스가 바깥 인스턴스와 독립적으로 존재할 수 있다면 정적 멤버 클래스로 만들어야한다. 

### 비정적 멤버 클래스
비정적 멤버 클래스의 인스턴스와 바깥 인스턴스의 관계는 멤버 클래스가 인스턴스화 될 때 확립. 
보통은 바깥 클래스의 인스턴스 메서드에서 비정적 멤버 클래스의 생성자를 호출할 때 자동으로 생성 -> 수동으로는 바깥 인스턴스의 클래스.new MemberClass(args)를 호출

- 비정적 멤버 클래스의 인스턴스 안에 만들어져 메모리 공간을 더 차지, 생성 시간도 더 걸림

어떤 클래스의 인스턴스를 감싸 마치 다른 클래스의 인스턴스처럼 보이게 하는 뷰로 사용

```java
public class HashMap<K,V> extends AbstractMap<K,V>
    implements Map<K,V>, Cloneable, Serializable {
    final class KeyIterator extends HashIterator
        implements Iterator<K> {
        public final K next() { return nextNode().key; }
    }
```

> 멤버 클래스에서 바깥 인스턴스에 접근할 일이 없다면 무조건 static을 붙여서 정적 멤버 클래스로 만들자. 

static 생략 -> 바깥 인스턴스로의 숨은 외부 참조 갖게됨
이 참조를 저장 -> 시간과 공간 소비 
GC가 바깥 클래스의 인스턴스를 수거하지 못하는 메모리 누수 발생
참조가 눈에 보이지 않아 문제의 원인을 찾지 어려움

```java
public class Example {

    private int value;
    class PublicEx{
        public void access() {
            value = 10;
        }
    }

    class PrivateEx{
        public void access() {
            value = 5;
        }
    }
}
```
![](https://velog.velcdn.com/images/rlfrkdms1/post/84eab885-3837-4d52-aa13-b58e3b4fd118/image.png)



### private 정적 멤버 클래스
흔히 바깥 클래스가 표현하는 객체의 한 부분을 나타낼 때 사용
Map을 생각했을 때 구현체는 모든 엔트리가 맵과 연관되어 있지만 엔트리의 메서드들은 맵을 직접 사용하지 않는다. 따라서 엔트리를 비정적 멤버 클래스로 표현하는 것은 낭비고, private 정적 멤버 클래스가 가장 알맞다. 

멤버 클래스가 public이나 protected 멤버라면 정적이냐 아니냐는 굉장히 큰문제다. 멤버 클래스 역시 공개 API가 되니, 혹시라도 향후 릴리스에서 static을 붙이면 하위호환성이 깨진다. 

### 익명 클래스

쓰이는 시점에 선언과 동시에 인스턴스가 만들어진다. 

상수 표현을 위해 초기화된 final 기본 타입과 문자열 필드만 가질 수 있다. 응용하는 데 제약이 많다. 또다름 주쓰임은 정적 팩터리 메서드를 구현할 때다. 
