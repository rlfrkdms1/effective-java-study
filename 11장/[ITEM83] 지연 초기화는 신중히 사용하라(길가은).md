# 지연 초기화

지연 초기화는 필드의 초기화 시점을 그 값이 처음 필요할 때까지 늦추는 기법이다. 

- 값이 쓰이지 않으면 초기화도 일어나지 않음
- 정적 필드와 인스턴스 필드 모두 사용
- 주로 최적화 용도
- 위험한 순환 문제 해결

> 필요할 때 까지는 하지 말라

지연 초기화를 사용하면 초기화 비용은 줄지만, 지연 초기화하는 필드에 접근하는 비용은 커진다. 

## 언제 필요할까?

해당 클래스의 인스턴스 중 그 필드를 사용하는 인스턴스의 비율이 낮은 반면 그 필드를 초기화하는 비용이 클 때

## 멀티 스레드 환경에서

> 대부분의 상황에서 일반적인 초기화가 지연 초기화보다 낫다. 

아래와 같은 메서드가 있다고 가정하자. 
```java
    private static FieldType computeFieldValue() {
        return new FieldType();
    }
```

### 일반적인 초기화 방법
아래는 일반적인 초기화 방법이다. 
```java
private final FieldType field = computeFieldValue();
```

### synchronized 지연 초기화
지연 초기화가 초기화 순환성을 깨뜨릴 것 같으면 synchronized를 단 접근자를 사용하자. 

```java
private FieldType field;

private synchronized FieldType getField(){
	if (field == null) 
    	field = computeFieldValue();
    return field;
}
```
위는 synchronized를 사용한 지연 초기화 방법이다. 

### 지연 초기화 홀더 클래스
> 성능 때문에 정적 필드를 지연 초기화해야 한다면 지연 초기화 홀더 클래스 관용구 사용하자. 

```java
private static class FieldHolder{
	static final FieldType field = computeFieldValue();
}

private static FiedlType getField() { return FieldHolder.field; }
```

getField가 처음 호출되는 순간 FieldHolder.field가 처음 읽히면서 클래스의 초기화를 촉발한다. 이는 동기화를 하지 않아 성능상의 저하가 없다. 

### 이중 검사 관용구 (double check)
> 성능 때문에 인스턴스 필드를 지연 초기화해야 한다면 이중검사 관용구를 사용하라

초기화된 필드에 접근할 때의 동기화 비용을 없애준다. 

동기화 없이 검사하고, 그다음 동기화하여 검사하는 방식이다. 

```java
    private volatile FieldType field4;

    private FieldType getField4() {
        FieldType result = field4;
        if (result != null)    // 첫 번째 검사 (락 사용 안 함)
            return result;

        synchronized(this) {
            if (field4 == null) // 두 번째 검사 (락 사용)
                field4 = computeFieldValue();
            return field4;
        }
    }
```
필드가 초기화된 후로는 동기화하지 않으므로 해당 필드는 반드시 volatile로 선언해야한다. 

result 변수는 필드를 한번만 읽는 것을 보장하는 역할을 한다. 

### 단일 검사 관용구 (single check)
이중 검사에서 두 번째 검사를 생략한 것이다. 

```java
    private volatile FieldType field5;

    private FieldType getField5() {
        FieldType result = field5;
        if (result == null)
            field5 = result = computeFieldValue();
        return result;
    }
```

모든 스레드가 필드의 값을 다시 계산해도 상관없고 필드의 타입이 long과 double을 제외한 다른 기본 타입이라면, 단일 검사의 필드 선언에서 volatile 한정자를 없애도 된다. 이는 racy single-check 관용구라 불린다. 보통은 거의 쓰이지 않는다. 

#### 출처 

이펙티브 자바 3/E
