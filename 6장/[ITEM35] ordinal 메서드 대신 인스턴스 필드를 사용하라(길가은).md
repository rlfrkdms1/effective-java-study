열거 타입은 해당 상수가 그 열거 타입에서 몇 번째 위치인지를 반환하는 `ordinal`이라는 메서드를 제공한다. 

합주단을 예로한 다음의 코드를 살펴보자. 각 위치는 합주단의 인원수를 가리킨다. 즉, 첫번째인 `SOLO`는 연주자가 1명인 것이다. 

```java
public enum Ensemble {
	SOLO, DUET, TRIO, QUARTET, QUINTET, SEXTET, SEPTET, OCTET, NONET, DECTET;
    
    public int numberOfMusicians() { return ordinal() + 1; }
```
동작은 하나 유지보수하기 정말 힘들다. 중간에 새로운 상수가 추가된다면, `numberOfMusicians()` 메서드는 더이상 사용하지 못한다. 또한, 중간에 상수끼리 위치가 뒤바껴도 마찬가지, 상수를 누락해도 마찬가지이다. 중간에 상수를 비울 수도 없어 더미 상수가 많아진다. 

따라서 이를 해결하기 위해선 **열거 타입 상수에 연결된 값을 ordinal 메서드로 얻지 말고, 인스턴스 필드에 저장하자** 

```java
public enum Ensemble {
    SOLO(1), DUET(2), TRIO(3), QUARTET(4), QUINTET(5),
    SEXTET(6), SEPTET(7), OCTET(8), DOUBLE_QUARTET(8),
    NONET(9), DECTET(10), TRIPLE_QUARTET(12);

    private final int numberOfMusicians;
    Ensemble(int size) { this.numberOfMusicians = size; }
    public int numberOfMusicians() { return numberOfMusicians; }
}
```

ordinal은 API문서에도 아래와 같이 쓰여져 있으니 용도외 사용하지 말아야한다. 
> 대부분 프로그래머는 이 메서드를 쓸 일이 없다. 이 메서드는 `EnumSet`과 `EnumMap`같이 열거 타입 기반의 범용 자료구조에 쓸 목적으로 설계되었다. 

#### 출처

이펙티브 자바 3/E
[이펙티브 자바 github](https://github.com/WegraLee/effective-java-3e-source-code/tree/master)
