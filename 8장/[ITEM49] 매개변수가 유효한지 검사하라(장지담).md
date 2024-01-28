# 매개변수 검사
> 매개변수 검사 : 메서드 멈체가 실해되기 전 매개변수를 확인하는 것 ex) 인덱스 값은 음수가 아니다. 객체 참조는 null이 아니다.

### 매개변수 검사를 제대로 하지 않으면 
1. 메서드 수행 중간에 모호한 예외를 던지며 실패
2. 메서드는 잘 수행되지만 잘못된 결과 반환
3. 메서드는 잘 수행되지만 실패 원자성 어김
> 실패 원자성 : 메서드 호출에 실패해도 객체는 이전 상태와 동일해야 한다

### 매개변수 값이 잘못되었을 때 던지는 예외를 문서화하자
- public, protected 메서드는 매개변수 값이 잘못되었을 때 전지는 예외를 문서화해야 한다.
- `@throws` javadoc 태그 사용

```java
    /**
     * 10을 num으로 나눈 값을 반환한다.
     * 
     * @param num 10을 나눌 수(0을 초과한 정수여야 한다)
     * @return 10/num
     * @throws IllegalArgumentException 0이하의 수로 나눌 수 없습니다
     */
    public int divideTen(int num){
        if (num<=0){
            throw new IllegalArgumentException("0이하의 수로 나눌 수 없습니다");
        }
        return 10/num;
    }
```

# requiredNonNull 메서드

```java
    public Integer divideTen(Integer num) {
        Integer checkedNum = Objects.requireNonNull(num, "null일 수 없다");
        return 10 / checkedNum;
    }
```
- null 검사를 지원하는 메서드다. 예외 메세지를 지정할 수 있으며, 값을 그대로 반환한다.
- 추가로 checkedFromIndexSize, checkFromToIndex, checkIndex 메서드들은 예외 메세지를 지정할 수 없고, 리스트 배열 전용이다.


# assert 사용 
- 공개되지 않은 private 메서드는 호출 상황을 통제할 수 있다.
- 단언문을 사용해 유효한 값만이 메서드에 넘겨지리라는 것을 보증하자

```java
    public Integer dividePositives(Integer a, Integer b) {
        assert a > 0;
        assert b > 0;
        return a / b;
    }
```
- 단언문은 조건이 무조건 참이라고 선언한다
- 실패하면 AssertionError를 던진다
- 런타임에 아무런 효과와 성능 저하가 없다

# 나중에 쓰려고 저장하는 매개변수의 유효성을 검사하라
- 문제가 발생한 지점과 원인이 멀어져 디버깅이 어려울 수 있다.
- 생성자

# 예외
- 유효성 검사 비용이 지나치게 높거나 실용적이지 않을 때
- 계산 과정에서 암묵적으로 검사가 시행될 때
  - 이 경우 API에서 명시한 예외와 다른 예외가 던져질 수 있는데, 예외 번역을 활용해 알맞은 예외로 바꿔주자
 

