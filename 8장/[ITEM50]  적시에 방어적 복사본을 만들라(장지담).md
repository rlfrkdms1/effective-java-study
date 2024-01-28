- java는 안전한 언어다. 그래도 방어적으로 프로그래밍 해야 한다.

```java
public final class Period {
    private final Date start;
    private final Date end;
    ... 이하생략
```
```java
        Date start = new Date();
        Date end = new Date();
        Period p = new Period(start, end);
        end.setYear(78);  // p의 내부를 변경했다!
```
- start와 end 필드는 final로 선언했지만, Date 자체가 가변이라 외부에서 변경할 수 있다.
- Date 대신 불변인 객체를 사용하면 된다. Instnat, LocalDateTime 등 ..

# 방어적 복사
- Date 처럼 가변인 객체를 참조하는 경우 **생성자에서 받은 가변 매개변수 각각을 방어적 복사하자.**
```java
    public Period(Date start, Date end) {
        this.start = new Date(start.getTime());
        this.end   = new Date(end.getTime());

        if (this.start.compareTo(this.end) > 0)
            throw new IllegalArgumentException(
                    this.start + "가 " + this.end + "보다 늦다.");
    }
```
- 생성자에서 값을 초기화 할 때 방어적 복사를 했다.
- 매개변수의 유효성을 검사하기 전에 방어적 복사본을 만들고, 이 방어적 복사본의 유효성을 검사했다.
  - 멀티스레딩 환경에서 매개변수의 유효성을 검사한 후 방어적 복사본을 만들면, TOCTOU 공격에 노출될 수 있다.
 > TOCTOU : 유효성을 검사한 후, 복사본을 만들기 전 짧은 순간에 원본 객체를 수정하는 공격
- 매개변수가 제 3자에 의해 확장될 수 있는 타입이라면 방어적 복사본을 만들 때 clone을 사용해서는 안된다.
  - 방어적 복사에 clone을 활용하지 않았다
  - Date는 final이 아니므로(상속가능하므로) clone이 Date가 정의한 게 아닐 수 있다. 즉 Date의 악의적인 하위 클래스일 수 있다. 
  - clone이 악의를 가진 하위 클래스의 인스턴스를 반환할 수 있다. (ITEM13에서 자세하게 다뤘었습니다)

```java
        start = new Date();ㅇ
        end = new Date();
        p = new Period(start, end);
        p.end().setYear(78);  // p의 내부를 변경했다!
```
- 하지만 접근자가 내부의 가변 정보를 직접 드러내 변경할 수 있다.

```java
    public Date start() {
        return new Date(start.getTime());
    }

    public Date end() {
        return new Date(end.getTime());
    }
```
- **접근자에서 방어적 복사본을 반환하자.**
- 접근자에서는 clone을 사용해도 된다. Date 객체임이 보장되기 때문이다.
- 그래도 복사 생성자 혹은 정적 팩터리를 쓰자.(이유 : ITEM13)

### 되도록 불변 객체들을 조합해 객체를 구성하자
- 방어적 복사는 성능 저하가 따른다.
- 항상 쓸 수 있는 것도 아니다.

# 예외상황
- 호출자가 컴포넌트 내부를 수정하지 않으리라 확신하면 방어적 복사를 하지 않아도 된다
- 때로는 메서드나 생성자의 매개변수로 넘기는 행위가 그 객체의 통제권을 명백히 이전함을 뜻한다. 이런 경우 방어적 복사를 할 필요가 없다. 다만 메서드를 호출하는 클라이언트는 객체를 수정하지 않는다고 약속해야 한다.
  - 이런 메서드나 생성자를 가진 클래스들은 악의적인 클라이언트의 공격에 취약하다. 따라서 방어적 복사를 생략해도 되는 상황은 아래와 같다.
    - 클래스와 클라이언트가 상호 신뢰할 수 있을 때
    - 불변식이 깨져도 클라이언트만 영향을 받을 때
  
  

  


