# 직렬화 프록시 패턴 
- Serializable을 구현한다는 것 자체가 생성자 이외의 방법으로 객체를 생성하게 하고 이것은 위험하다. **직렬화 프록시 패턴**이 이 위험을 크게 줄여준다.

```java
// 방어적 복사를 사용하는 불변 클래스
public final class Period implements Serializable {
    private final Date start;
    private final Date end;

    /**
     * @param start 시작 시각
     * @param end   종료 시각; 시작 시각보다 뒤여야 한다.
     * @throws IllegalArgumentException 시작 시각이 종료 시각보다 늦을 때 발생한다.
     * @throws NullPointerException     start나 end가 null이면 발행한다.
     */
    public Period(Date start, Date end) {
        this.start = new Date(start.getTime());
        this.end = new Date(end.getTime());
        if (this.start.compareTo(this.end) > 0)
            throw new IllegalArgumentException(
                    start + "가 " + end + "보다 늦다.");
    }

    public Date start() {
        return new Date(start.getTime());
    }

    public Date end() {
        return new Date(end.getTime());
    }

    public String toString() {
        return start + " - " + end;
    }


    // 코드 90-1 Period 클래스용 직렬화 프록시 (479쪽)
    private static class SerializationProxy implements Serializable {
        private final Date start;
        private final Date end;

        SerializationProxy(Period p) {
            this.start = p.start;
            this.end = p.end;
        }

        private Object readResolve() {
            return new Period(start, end);
        }

        private static final long serialVersionUID =
                234098243823485285L; // 아무 값이나 상관없다. (아이템 87)
    }

    // 직렬화 프록시 패턴용 writeReplace 메서드 (480쪽)
    private Object writeReplace() {
        return new SerializationProxy(this);
    }

    // 직렬화 프록시 패턴용 readObject 메서드 (480쪽)
    private void readObject(ObjectInputStream stream)
            throws InvalidObjectException {
        throw new InvalidObjectException("프록시가 필요합니다.");
    }
}
```
- 중첩 클래스가 바깥 클래스의 직렬화 프록시다.
  - 중첩 클래스는 바깥 클래스의 논리적 상태를 표현하는 private static 클래스다.
  - 중첩 클래스의 생성자는 하나여야 하고 바깥 클래스를 매개변수로 받아서 데이터를 복사한다.
  - 직렬화 클래스의 기본 직렬화 형태를 바깥 클래스의 직렬화 형태로 사용한다. 
- 바깥 클래스에 writeReplace를 추가한다. 
  - 이것은 복사해 사용하면 된다.(범용적인 메서드로 모든 클래스에서 똑같이 사용한다)
  - 이 메서드는 직렬화 시스템이 바깥 클래스 인스턴스 대신 SerializationProxy 인스턴스를 반환하게 한다.
- writeReplace 메서드로 인해 바깥 클래스의 인스턴스를 생성할 수 없지만 공격자가 시도하는 것을 막기 위해 readObject에서 예외를 던지게 한다.
- readResolve는 역직렬화 시에 직렬화 시스템이 직렬화  프록시를 다시 바깥 클래스 인스턴스로 변환하게 해준다.
  - 공개 API 즉 생성자, 정적팩터리 같은 일반 인스턴스를 만들 때와 같은 방법으로 객체를 만든다.
  - 불변식 검사 등 (이미 생성자나 정적팩터리에서 하고 있을) 에 대해 신경 쓸 필요가 없다. 
- 직렬화 프록시 패턴은 가짜 바이트 스트림 공격과 내부 필드 탈취 공격을 프록시 수준에서 차단해준다.
- 직렬화 클래스는 Period의 필드를 final로 선언해도 되므로 불변 클래스로 만들 수 있다.
- 역직렬화한 인스턴스와 원래 직렬화된 인스턴스의 클래스가 달라도 정상 동작한다
  - EnumSet은 public 생성자 없이 정적 팩터리만 제공한다. 원소가 64개 이하면 RegularEnumSet을, 초과면 JumboEnumSet을 사용한다.
  - 원소 64개짜리 열거 타입을 가진 EnumSet을 직렬화하고 원소 5개를 추가한 후 역직렬화한다면?
  - 직렬화 할 때 : RegularEnumSet, 역직렬화 할 때 : JumboEnumSet, EnumSet은 직렬화 프록시 패턴을 사용해 이렇게 구현했다. 
```java
    private static class SerializationProxy<E extends Enum<E>>
        implements java.io.Serializable
    {

        private static final Enum<?>[] ZERO_LENGTH_ENUM_ARRAY = new Enum<?>[0];

        /**
         * The element type of this enum set.
         *
         * @serial
         */
        private final Class<E> elementType;

        /**
         * The elements contained in this enum set.
         *
         * @serial
         */
        private final Enum<?>[] elements;

        SerializationProxy(EnumSet<E> set) {
            elementType = set.elementType;
            elements = set.toArray(ZERO_LENGTH_ENUM_ARRAY);
        }

        /**
         * Returns an {@code EnumSet} object with initial state
         * held by this proxy.
         *
         * @return a {@code EnumSet} object with initial state
         * held by this proxy
         */
        @SuppressWarnings("unchecked")
        private Object readResolve() {
            // instead of cast to E, we should perhaps use elementType.cast()
            // to avoid injection of forged stream, but it will slow the
            // implementation
            EnumSet<E> result = EnumSet.noneOf(elementType);
            for (Enum<?> e : elements)
                result.add((E)e);
            return result;
        }

        private static final long serialVersionUID = 362491234563181265L;
    }
```
- 역직렬화 할 때 정적 팩터리 메서드 noneOf로 EnumSet을 생성하고 이 메서드에서는 원소 수에 따라 알맞은 EnumSet을 만든다.

# 한계점 
- 클라이언트가 멋대로 확장할 수 있는 클래스에는 적용할 수 없다
- 객체 그래프에 순환이 있는 클래스에도 적용할 수 없다
- 느려진다

> 가능한 한 직렬화 프록시 패턴을 사용하자. 불변식을 안정적으로 직렬화해주는 가장 쉬운 방법이다. 
