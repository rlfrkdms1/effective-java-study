equals를 재정의 한 클래스 모두에서 hashcode도 재정의해야한다. 그렇지 않으면 HashMap,HashSet 같은 컬렉션의 원소로 사용할 때 문제가 발생한다.

# hashcode 재정의 규약
- equals 비교에 사용되는 정보가 변경되지 않았다면, 애플리케이션이 실행되는 동안 그 객체의 hashcode는 호출할 때 마다 같은 값을 반환해야 한다.
- equals가 두 객체를 같다고 판단했다면 두 객체의 hashCode는 같은 값을 반환해야 한다.
- equals가 두 객체를 다르다고 판단했다고 hashCode 반환값이 다를 필요는 없다. 하지만 다른 값을 반환해야 해시테이블의 성능이 좋아진다.

eqauls를 논리적 동치를 비교하도록 재정의하고 hashcode는 재정의하지 않으면 두번째 규약을 어기게 된다. Object의 hashcode는 논리적 동치더라도 물리적 동치가 아니면 다른 값을 반환하기 때문이다.

```java
    @Override
    public int hashCode() {
        return 16;
    }
```
- 적법하지만 모든 객체의 hashCode 값이 같으므로 세번째 규약을 지키지 않고 있다(필수가 아니므로)
- 해시테이블 충돌로 성능이 매우 떨어진다.  

# hashcode 재정의 방법
1. int result를 c로 초기화한다. c는 객체의 첫번째 핵심 필드(equals 비교에 사용되는 필드)를 계산한 hashcode다.
2. 객체의 나머지 핵심 필드 f 각각에 대해 아래 작업을 수행한다.
   1) 해당 필드의 해시코드 c를 계산한다.
   2) result = 31*result + c;로 result를 갱신한다.
3. result를 반환한다.

### 필드별 해시코드 계산 방법
- 기본 타입 : Type.hashcode(f), 여기서 Type은 박싱 클래스
- 참조 타입 필드면서 이 클래스의 equals가 이 필드의 equals를 호출한다면 이 필드의 hashCode를 재귀적으로 호출한다. 필드값이 null이면 0을 사용한다.(전통적으로 0 사용)
- 필드가 배열이라면 핵심 원소 각각을 별도 필드 취급한다. 배열에 핵심 원소가 하나도 없다면 0을 사용한다. (다른 상수도 무관) 모든 원소가 핵심 원소면 Arrays.hashCode를 사용한다.

```java
    @Override
    public int hashCode() {
        int result = Integer.hashCode(age);
        result = 31 * result + Integer.hashCode(battery);
        result = 31 * result + Integer.hashCode(power);
        return result;
    }
```

- 파생 필드(다른 필드로부터 계산할 수 있는 필드)는 제외해도 된다.
- equals에 사용되지 않은 필드는 반드시 제외해야 한다.

# Object.hash
```java
    @Override
    public int hashCode() {
        return Objects.hash(age, battery, power);
    }
```
- Objects.hash로 단 한 줄로 hashCode를 작성할 수 있다.
- 앞의 방법 속도가 느리다. 성능에 민감하지 않은 상황에서만 사용하자.

# 캐싱
```java
    private int hashCode = 0;

    @Override
    public int hashCode() {
        int result = hashCode;
        if (result==0){
            result = Objects.hash(age, battery, power);
        }
        return result;
    }
```
- 불변 클래스이고 hashCode 계산 비용이 크다면 매번 새로 생성하기보다 캐싱할 수 있다.
- 인스턴스가 만들어 질 때 해시코드를 계산해두거나 지연 초기화 한다.
  - hashCode 필드 값을 흔히 생성되는 해시코드 값과 다른 값으로 초기화해놓고 hashCode 메서드를 호출하면 hashCode 필드를 검사한다. 초기값이라면(아직 해시값은 계산하지 않았다면) 계산해서 반환하고 초기값이 아니라면(이미 계산했다면) 재사용한다.

# 주의점
- 해시코드를 계산할 때는 핵심 필드를 생략해서는 안된다. 해시테이블의 성능 저하로 이어질 수 있다.
- hashCode가 반환하는 값의 생성 규칙을 API 사용자에게 자세히 공표하지 말자. 그래야 클라이언트가 이 값에 의지하지 않게 되고 추후에 계산 방식을 바꿀 수 있다.
