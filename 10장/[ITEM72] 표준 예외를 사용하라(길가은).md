# 표준 예외
표준 예외를 재사용하면 아래와 같은 장점이 있다. 
- API가 다른 사람이 익히고 사용하기 쉬워진다. 
    - 이미 익숙한 규약을 따르기 때문
- 낯선 예외를 사용하지 않아 읽기 쉽다. 
- 예외 클래스 수가 적을수록 메모리 사용량도 줄고 클래스를 적재하는 시간도 적게 걸린다. 

## 표준 예외들

|예외|주요 쓰임|
|-|-|
|`IllegalArgumentException`| 허용하지 않는 값이 인수로 건네졌을 때 ex) 반복횟수를 지정하는 매개변수에 음수를 건넬 때|
|`IllegalStateException`|객체가 메서드를 수행하기에 적절하지 않은 상태일 때 ex) 제대로 초기화되지 않은 객체를 사용하려 할 때|
|`NullPointerException`|null을 허용하지 않는 메서드에 null을 건넸을 때|
|`IndexOutOfBoundsException`|인덱스가 범위를 넘어섰을 때|
|`ConcurrentModificationException`|허용하지 않는 동시 수정이 발견됐을 때, 문제가 생길 가능성 정도를 알려줌|
|`UnsupportedOperationException`|호출한 메서드를 지원하지 않을 때 ex) List 구현체에 remove 메서드를 호출할 때|

>`Exception`, `RuntimeException`, `Throwable`, `Error`는 직접 재사용하지 말자. 

이 예외들은 추상 클래스라고 생각해야한다. 여러 성격의 예외들을 포괄하는 클래스이므로 안정적으로 테스트할 수 없다. 

API문서를 참고해 예외가 어떤 상황에서 던져지는지 꼭 확인하자. 예외의 이름뿐 아니라 예외가 던져지는 맥락도 부합할 때만 재사용한다. 

> 인수 값이 무엇이었든 어차피 실패했을거라면 `IllegaStateException`을, 그렇지 않으면 `IllegalArgumentException`을 던지자.


#### 출처

이펙티브 자바 3/E

