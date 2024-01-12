# Comparable이란?
> Comparable 인터페이스의 compareTo는 Object의 equals랑 비슷한데, 동치성 비교에 더해 순서까지 비교할 수 있다.

- 순서가 명확한 클래스에 정렬, 비교 기능을 넣어야 한다면 Comparable을 구현하자

# compareTO 메서드 일반 규약
이 객체와 주어진 객체의 순서를 비교한다. 이 객체가 주어진 객체보다 작으면 음의 정수, 같으면 0, 크면 양의 정수를 반환한다. 
이 객체와 비교할 수 없는 타입의 객체가 주어지면 ClassCastException을 던진다. 
아래 규약에서 sgn은 부호 함수로 sgn(식)에서 식이 음수,0,양수 일 때 -1,0,1을 반환한다.

1. `sgn(x.compareTo(y)) == -sgn(y.compareTo(x))`여야 한다. 
2. Comparable을 구현한 클래스는 추이성을 보장해야 한다. 
    - `x.compareTo(y)>0 && y.compareTo(z)>0` 이면 `x.compareTo(z) >0`
3. Comparable을 구현한 클래스는 모든 z에 대해 `x.compareTo(y) ==0` 이면 `sgn(x.compareTo(z)) == sgn(y.compareTo(z))`이다. 
4. `(x.compareTo(y) == 0) == (x.equals(y))`
    - 필수는 아니지만 지키는 게 좋다 
    - Comparable을 구현하고 이 권고를 지키지 않으면 클래스의 순서가 equals와 일관되지 않다.

compareTo 규약을 지키지 않으면 비교를 활용하는 클래스와 어울리지 못한다. 예를 들자면 TreeSet, TreeMap, Collections, Arrays등이 있다. 

---

- compareTo는 equals와 달리 타입이 다른 객체를 신경쓸 필요가 없다
  - 타입이 다른 객체가 주어지면 ClassCastException을 던져도 되며, 대부분 그렇게 한다
- 규약 1,2,3은 compareTo로 수행하는 동치성 검사도 equals와 같이 반사성, 대칭성, 추이성을 충족해야 한다는 뜻이다. 
- equals와 주의사항도 마찬가지로 기존 클래스를 확장한 구체 클래스에서 새로운 필드를 추가했다면 compareTo 규약을 지킬 수 없다.
  - 상속 대신 컴포지션을 사용하도록 해 해결한다
- compareTo로 수행한 동치성 테스트의 결과가 equals와 같아야 한다는 마지막 규약을 지키지 않으면 Collection, Set, Map 등에 정의된 동작과 어긋나게 된다

# compareTo 작성요령
### 입력 인수의 타입을 확인하거나 형변환 할 필요가 없다
  - 컴파일에러로 잡을 수 있다.
  - null을 넣어 호출하면 NPE를 던져야 한다.
 
### 객체 참조 필드를 비교하려면 compareTo를 재귀적으로 호출한다.
- 필드가 Comparable을 구현하지 않았거나, 표준이 아닌 순서로 비교해야 한다면 Comparator(비교자)를 사용한다
  -  Comparator는 직접 만들거나 자바가 제공하는 것을 사용한다.
```java
public final class CaseInsensitiveString implements Comparable<CaseInsensitiveString> {
    // 자바가 제공하는 비교자를 사용해 클래스를 비교한다.
    public int compareTo(CaseInsensitiveString cis) {
        return String.CASE_INSENSITIVE_ORDER.compare(s, cis.s);
    }
    ...
}
```
- 위 예시에서 `CaseInsensitiveString implements Comparable<CaseInsensitiveString>`는 CaseInsensitiveString의 참조는 CaseInsensitiveString 참조와만 비교할 수 있다는 뜻이고, Comparable을 구현할 때 일반적으로 이렇게 한다.

### 정적 메서드 compare를 사용하자
```java
Integer.compare(age,robot.age);
```
  - compareTo 메서드에서 관계 연산자 <, >를 사용하는 방식은 거추장스럽고 오류를 유발한다.
  - 박싱된 기본 타입 클래스의 정적 메서드 compare를 사용하자.

### 기본 타입 필드가 여럿일 때 비교자
```java
    public int compareTo(PhoneNumber pn) {
        int result = Short.compare(areaCode, pn.areaCode);
        if (result == 0)  {
            result = Short.compare(prefix, pn.prefix);
            if (result == 0)
                result = Short.compare(lineNum, pn.lineNum);
        }
        return result;
    }
```
- 필드가 여러개라면 핵심적인 필드부터 비교하자
- 비교 결과가 0이 아니라면 (순서가 결정되면) 거기서 끝이다. 그 결과를 반환하자.

### 비교자 생성 메서드를 활용한 비교자
- Comparator(비교자) 생성 메서드를 통해 메서드 체이닝 방식으로 비교자를 생성할 수 있다. compareTo 구현에 활용할 수 있다.
- 간결하지만 약간의 성능 저하(약 10%)가 발생한다.
```java
    private static final Comparator<PhoneNumber> COMPARATOR =
            comparingInt((PhoneNumber pn) -> pn.areaCode)
                    .thenComparingInt(pn -> pn.prefix)
                    .thenComparingInt(pn -> pn.lineNum);

    public int compareTo(PhoneNumber pn) {
        return COMPARATOR.compare(this, pn);
    }
```
- comparingInt
  - 객체 참조를 int 타입 키에 매핑하는 키 추출 함수를 인자로 받아 그 키를 기준으로 순서를 정하는 비교자를 반환하는 정적 메서드 
  - 예시에서 람다를 인수로 받음, 람다는 areaCode로 순서를 정하는 Comparator<PhoneNumber> 반환 
  - 람다 입력 인수 타입을 명시함
- thenComparingInt
  - Comparator의 인스턴스 메서드, int 키 추출자 함수를 입력받아 다시 비교자 반환. 이 비교자는 첫 번재 비교자를 적용한 후 다음 키로 추가 비교 수행 
  - 원하는 만큼 연달아 호출할 수 있다.
 
```java
@RequiredArgsConstructor
public class Robot implements Comparable<Robot> {

    private static final Comparator<Robot> COMPARATOR = Comparator.comparing((Robot robot) -> robot.arm)
            .thenComparing(robot -> robot.leg)
            .thenComparing(robot -> robot.head);

    private final Arm arm;
    private final Leg leg;
    private final Head head;

    @Override
    public int compareTo(Robot o) {
        return COMPARATOR.compare(this, o);
    }
}
```
객체 참조용 비교자 생성 메서드도 comparing과 thenComparing도 있다. Arm, Leg, Head는 각각 Comparable을 구현하고 있다. 

# 주의사항
```java
    static Comparator<Robot> hashCodeOrder = new Comparator<Robot>() {
        @Override
        public int compare(Robot o1, Robot o2) {
            return o1.hashCode() - o2.hashCode();
        }
    };
```
- 값의 차로 compareTo나 compare를 구현하는 경우가 있다.
- 정수 오버플로를 일으키거나 IEEE754 부동소수점 계산방식에 따른 오류를 낼 수 있다. 
```java
    static Comparator<Robot> hashCodeOrder = new Comparator<Robot>() {
        @Override
        public int compare(Robot o1, Robot o2) {
            return Integer.compare(o1.hashCode(),o2.hashCode());
        }
    };
    
    static Comparator<Robot> hashCodeOrder = 
    	Comparator.comparingInt(o->o.hashCode());
```
위 두 방식 중 하나를 대신 사용하자.


