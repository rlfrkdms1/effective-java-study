# 기본 직렬화 형태
- 객체의 기본 직렬화 형태는 그 객체를 루트로 하는 객체 그래프의 물리적 모습을 인코딩한다.
- 객체가 포함한 데이터, 접근할 수 있는 객체, 객체들이 연결된 위상
- 고민해보고 괜찮을 때만 기본 직렬화 형태를 사용하자. 유연성, 성능, 정확성 측면에서 고민 후 사용하자

```java
    public static void main(String[] args) {
        Name name = new Name("as", "df", "cd");
        String fileName = "Name.ser";

        try (
                FileOutputStream fos = new FileOutputStream(fileName);
                ObjectOutputStream out = new ObjectOutputStream(fos);
        ) {
            // 직렬화
            out.writeObject(name);
        } catch (IOException e) {

        }

        try (
                FileInputStream fis = new FileInputStream(fileName);
                ObjectInputStream in = new ObjectInputStream(fis);
        ) {
            // 역직렬화
            Name deserialized = (Name) in.readObject();
        } catch (FileNotFoundException e) {
            throw new RuntimeException(e);
        } catch (IOException e) {
            throw new RuntimeException(e);
        } catch (ClassNotFoundException e) {
            throw new RuntimeException(e);
        }
    }
```
- 직렬화, 역직렬화는 Serializable 을 구현한 클래스에 대해 ObjectOutputStream의 writeObject와 ObjectInputStream의 readObject를 호출해서 진행할 수 있다. 

# 객체의 물리적 표현과 논리적 내용이 같다면 기본 직렬화 형태라도 무방하다. 
- 이상적인 직렬화 형태라면 물리적 모습과 독립된 논리적 모습만을 표현해야 한다.
- 물리적 표현 : 코드로 어떻게 구현했는지, 논리적 내용 : 실제 의미하는 내용

```java
public class Name implements Serializable {

    /**
     * 성, null이 아니어야 함 
     * @serial 
     */
    private final String lastName;
    /**
     * 이름, null이 아니어야 함 
     * @serial
     */
    private final String firstName;
    /**
     * 중간이름
     * @serial
     */
    private final String middleName;

    // 이하생략 ..
}
```
- 사람의 이름을 나타내는 이러한 클래스는 물리적 표현과 논리적 내용이 같으므로 기본 직렬화 형태를 사용해도 괜찮다
- 기본 직렬화 형태가 적합하다고 판단했더라도 불변식 보장과 보안을 위해 readObject 메서드(역직렬화에 사용하는 메서드)를 제공해야 할 때가 많다. 예를 들어 firstName, lastName은 null이 아니어야한다는 불변식을 지키고 싶으면 readObject에서 보장할 수 있다.
- private 필드더라도 직렬화에 속하므로 공개 API에 속해 문서화 주석이 달려있다. @serial 태그를 달면 직렬화 형태를 설명하는 javadoc 페이지에 기록된다. 

### 객체의 논리적 표현 - 물리적 표현이 차이가 클 때 기본 직렬화 표현을 사용하면 문제점

```java
public final class StringList implements Serializable {
    private transient int size   = 0;
    private transient Entry head = null;

    private static class Entry implements Serializable{
        String data;
        Entry  next;
        Entry  previous;
    }
...
}
```
- 문자열 리스트를 표현하는 StringList 클래스는 물리적으로 문자열들을 양방향 연결 리스트로 연결했고 논리적으로는 일련의 문자열을 표현한다. 따라서 직렬화 형태에 적합하지 않은 예다.
객체의 논리적 표현 - 물리적 표현이 차이가 클 때 기본 직렬화 표현을 사용하면 4가지 문제가 발생한다
1. 공개 API 가 현재의 내부 표현 방식에 영구히 묶인다

StringList.Entry가 공개 API가 되어 버리고 다음 릴리스에서 내부 표현 방식을 바꾸더라도 여전히 연결 리스트 관련 코드를 제거할 수 없다 

2. 너무 많은 공간을 차지할수 있다.

연결 리스트의 모든 엔트리, 연결 정보까지 직렬화하지만, 이것은 내부 구현에 해당하니 직렬화에 포함할 필요가 없다. 

3. 시간이 너무 많이 걸릴 수 있다.

객체 그래프 위상에 관한 정보가 없으니 직렬화 로직이 그래프를 직접 순회해 볼 수 밖에 없다 
> 객체 그래프 : 객체들 간 관계를 나타낸 구조

4. 스택 오버플로를 일으킬 수 있다.
기본 직렬화 과정은 객체 그래프를 재귀 순회하는데, 중간 정도 크기 객체에서도 스택 오버플로가 발생할 수 있으며 이것이 매번 달라진다.

### 합리적인 직렬화 구조 
- StringList의 물리적인 상세 표현은 배제하고 논리적인 구성만 담아야 한다
- 단순히 List가 포함된 문자열의 개수를 적은 다음, 문자열들을 나열하는 수준이면 된다.

```java
public final class StringList implements Serializable {
    private transient int size   = 0;
    private transient Entry head = null;

    // 이제는 직렬화되지 않는다.
    private static class Entry {
        String data;
        Entry  next;
        Entry  previous;
    }

    // 지정한 문자열을 이 리스트에 추가한다.
    public final void add(String s) {  }

    /**
     * 이 {@code StringList} 인스턴스를 직렬화한다.
     *
     * @serialData 이 리스트의 크기(포함된 문자열의 개수)를 기록한 후
     * ({@code int}), 이어서 모든 원소를(각각은 {@code String})
     * 순서대로 기록한다.
     */
    private void writeObject(ObjectOutputStream s)
            throws IOException {
        s.defaultWriteObject();
        s.writeInt(size);

        // 모든 원소를 올바른 순서로 기록한다.
        for (Entry e = head; e != null; e = e.next)
            s.writeObject(e.data);
    }

    private void readObject(ObjectInputStream s)
            throws IOException, ClassNotFoundException {
        s.defaultReadObject();
        int numElements = s.readInt();

        // 모든 원소를 읽어 이 리스트에 삽입한다.
        for (int i = 0; i < numElements; i++)
            add((String) s.readObject());
    }

    // 나머지 코드는 생략
}
```
- writeObjet와 readObject가 직렬화 형태를 처리한다
- transient 한정자는 해당  필드가 기본 직렬화 형태에 포함되지 않는다는 뜻이다. 
- 모든 필드가 transient더라도 writeObject, readObject에서 각각 가장 먼저 defaultWriteObject와 defaultReadObject를 호출한다. 이렇게 해야 향후 릴리스에서 transient가 아닌 필드가 추가되더라도 상호 호환된다.
- 메서드의 @serialData는 위에서 설명한 필드의 @serial과 같다
- 이 방식은 개선 전에 비해 절반 정도의 공간을 차지하고 스택 오버플로가 발생하지 않는다.

# 세부 구현에 따라 불변식이 달라지는 경우 기본 직렬화를 사용하면 정확성이 깨질 수 있다 
- StringList에서 기본 직렬화를 사용하더라도 객체를 직렬화 후 역직렬화하면 원래 객체를 불변식까지 포함해 복원해낸다는 점에서 정확하다.
- 세부 구현에 따라 불변식이 달라지는 경우 기본 직렬화를 사용하면 정확성이 깨질 수 있다
- 해시테이블은 물리적으로는 키-값 엔트리들을 담은 해시 버킷을 나열한 형태다.
-  어떤 엔트리를 어떤 버킷에 다믈 지는 키에서 구한 해시코드가 결정하는데 그 계산 방식은 구현에 따라 달라질 수 있고 계산할 때마다 달라질수 있다.
-  해시테이블에 기본 직렬화를 사용하면 심각한 버그로 이어질 수 있다. 해시테이블을 직렬화하고 역직렬화하면 불변식이 심각하게 훼손된 객체가 많이 생길 수 있다.

# transient
- 해당 객체의 논리적 상태와 무관한 필드라고 확신할 때만 transient를 생략해야 한다
- transient 필드들은 기본 직렬화를 사용하면 기본값으로 초기화된다.
  - 참조 필드 : null, 숫자 타입 : 0, boolean : false
  - readObject에서 defaultReadObject를 호출한 후 해당 필드를 원하는 값으로 복원하자(ITEM88)
  - 혹은 값을 처음 사용할 때 초기화 할 수도 있다

# 객체의 전체 상태를 읽는 메서드에 적용해야 하는 동기화 메커니즘을 직렬화에도 적용해야 한다.
모든 메서드가 synchronized로 선언되어 스레드 안전하게 만든 객체라면 writeObject도  synchronized로 선언해야 한다 


# 어떤 직렬화 형태를 택하든 직렬화 가능 클래스 모두에 직렬 버전 UID를 명시적으로 부여하자 
`privat static final long serialVersionUID = <무작위로 고른 long 값>;`
- UID가 일으키는 호환성 문제가 사라진다
- 성능도 빨라진다
- 고유할 필요도 없고 아무 생가나는 값을 사용해도 된다.
- 호환성을 끊으려는 경우가 아니면 UID를 수정하지 말자.
