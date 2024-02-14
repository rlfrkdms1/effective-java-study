# readObject는 사실상 또 다른 public 생성자이므로 주의를 기울이자
- 인수가 유효한지 검사하자
- 필요하다면 매개변수를 방어적으로 복사하자
- 매개변수로 바이트 스트림을 받는 생성자이므로 임의로 생성한 바이트 스트림을 건네 비정상적인 객체를 만들 수 있다. 따라서 readObject에서 defaultReadObject를 호출한 후 역직렬화된 객체가 유효한지 검사해야 한다.
- 유효성 검사에 실패하면 InvalidObjectExcepton을 던지자

```java
public final class Period {
    private final Date start;
    private final Date end;

    /**
     * @param  start 시작 시각
     * @param  end 종료 시각. 시작 시각보다 뒤여야 한다.
     * @throws IllegalArgumentException 시작 시각이 종료 시각보다 늦을 때 발생한다.
     * @throws NullPointerException start나 end가 null이면 발생한다.
     */
    public Period(Date start, Date end) {
        if (start.compareTo(end) > 0)
            throw new IllegalArgumentException(
                    start + "가 " + end + "보다 늦다.");
        this.start = start;
        this.end   = end;
    }

    public Date start() {
        return start;
    }
    public Date end() {
        return end;
    }

    public String toString() {
        return start + " - " + end;
    }

    // 나머지 코드 생략
}
```
```java
    // 정상적이지 않은 바이트스트림
    private static final byte[] serializedForm = {
        (byte)0xac, (byte)0xed, 0x00, 0x05, 0x73, 0x72, 0x00, 0x06,
        0x50, 0x65, 0x72, 0x69, 0x6f, 0x64, 0x40, 0x7e, (byte)0xf8,
        0x2b, 0x4f, 0x46, (byte)0xc0, (byte)0xf4, 0x02, 0x00, 0x02,
        0x4c, 0x00, 0x03, 0x65, 0x6e, 0x64, 0x74, 0x00, 0x10, 0x4c,
        0x6a, 0x61, 0x76, 0x61, 0x2f, 0x75, 0x74, 0x69, 0x6c, 0x2f,
        0x44, 0x61, 0x74, 0x65, 0x3b, 0x4c, 0x00, 0x05, 0x73, 0x74,
        0x61, 0x72, 0x74, 0x71, 0x00, 0x7e, 0x00, 0x01, 0x78, 0x70,
        0x73, 0x72, 0x00, 0x0e, 0x6a, 0x61, 0x76, 0x61, 0x2e, 0x75,
        0x74, 0x69, 0x6c, 0x2e, 0x44, 0x61, 0x74, 0x65, 0x68, 0x6a,
        (byte)0x81, 0x01, 0x4b, 0x59, 0x74, 0x19, 0x03, 0x00, 0x00,
        0x78, 0x70, 0x77, 0x08, 0x00, 0x00, 0x00, 0x66, (byte)0xdf,
        0x6e, 0x1e, 0x00, 0x78, 0x73, 0x71, 0x00, 0x7e, 0x00, 0x03,
        0x77, 0x08, 0x00, 0x00, 0x00, (byte)0xd5, 0x17, 0x69, 0x22,
        0x00, 0x78
    };

    public static void main(String[] args) {
        Period p = (Period) deserialize(serializedForm);
        System.out.println(p);
    }
```
- 위 Period 클래스에서 기본 직렬화를 사용한다면 임의의 바이트 스트림을 serializedForm을 전달해서 start가 end보다 늦은 Period 객체를 만들 수 있다. 불변식이 깨진다.

```java
    private void readObject(ObjectInputStream s) throws IOException, ClassNotFoundException {
        s.defaultReadObject();
        if (start.compareTo(end) > 0) {
            throw new InvalidObjectException(start + "가 " + end + " 보다 늦다");
        }
    }
```
- 따라서 위와 같이 역직렬화환 객체가 유효한지 검사할 수 있다.

```java
public class MutablePeriod {
    
   public final Period period;
   public final Date start;
   public final Date end;

   public MutablePeriod() {
       try {
           ByteArrayOutputStream bos = new ByteArrayOutputStream();
           ObjectOutputStream out = new ObjectOutputStream(bos);

           // 불변식을 유지하는 Period 를 직렬화한다.
           out.writeObject(new Period(new Date(), new Date()));

           /*
            * bos 값에 악의 적인 바이트스트림을 주입한다.
            */
           byte[] ref = { 0x71, 0, 0x7e, 0, 5 }; // 악의적인 바이트스트림
           bos.write(ref);                       // start 필드
           ref[4] = 4;                           // 악의적인 바이트스트림
           bos.write(ref);                       // end 필드

           // 역직렬화 과정에서 Period 객체의 Date 참조를 훔친다.
           ObjectInputStream in = new ObjectInputStream(new ByteArrayInputStream(bos.toByteArray()));
           
           period = (Period) in.readObject();
           start  = (Date) in.readObject();
           end    = (Date) in.readObject();
           
       } catch (IOException | ClassNotFoundException e) {
           throw new AssertionError(e);
       }
   }
}
```
```java
public static void main(String[] args) {
    MutablePeriod mp = new MutablePeriod();
    Period p = mp.period;
    Date pEnd = mp.end;

    pEnd.setYear(78);      // end 필드를 80년으로 수정한다.
    System.out.println(p);
        
    pEnd.setYear(69);      // end 필드를 60년으로 수정한다.
    System.out.println(p);
    }
```

- 유효한 Period 인스턴스에서 시작된 바이트 스트림 끝에 Date 필드로의 참조를 추가하면 가변 Period 인스턴스를 만들 수 있다.
- Period는 불변식을 유지한 채 생성했지만 의도적으로 내부 값을 수정할 수 있다.
- Period가 불변이라고 가정한 채 사용하는 클래스에 넘겨 보안 문제를 일으킬 수 있다.
- Period의 readObject가 방어적 복사를 충분히 하지 않아서 발생한 문제다.

```java
private void readObject(ObjectInputStream s) throws IOException, ClassNotFoundException {
    s.defaultReadObject();

    // 가변 요소들을 방어적으로 복사한다.r
    start = new Date(start.getTime());
    end   = new Date(end.getTime());

    // 불변식을 만족하는지 검사한다.
    if (start.compareTo(end) > 0) {
        throw new InvalidObjectException(start + "가 " + end + "보다 늦다.");
    }
}
```
- 객체를 역직렬화할때는 클라이언트가 소유해서는 안되는 객체 참조를 갖는 필드를 반드시 방어적으로 복사하자
- 방어적 복사를 유효성 검사보다 앞서 수행했으며 Date의 clone 메서드는 사용하지 않았다(ITEM50)
- 필드에서 final을 제거해야 하지만, 공격에 노출되는 것보다는 낫다

# 기본 readObject를 써도 좋을지 판단하는 방법
- transient 필드를 제외한 모든 필드의 값을 매개변수로 받아 유효성 검사 없이 필드에 대입하는 public 생성자를 추가해도 괜찮다면 사용하고 아니라면 커스텀 readObject를 만들어 유효성 검사를 수행하자
- 혹은 직렬화 프록시 패턴(ITEM90)을 사용할 수 있다

---

# 주의점
- 생성자와 마찬가지로 readObject도 직, 간접적으로 재정의 가능 메서드를 호출해서는 안된다
