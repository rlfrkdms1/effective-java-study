문자열을 쓰지 않아야 할 상황이 있다

# 문자열은 다른 값 타입을 대신하기에 적합하지 않다
- 파일, 네트워크, 키보드 입력을 받을 때 문자열을 사용하는 경우가 많지만 진짜 문자열일때만 그렇게 하자
- 수치형이라면 int, float, BigInteger등 적당한 수치 타입으로, 예/아니오 형태라면 boolean으로 변환하자
- 적절한 값 타입이 있다면 그것으로 변환하고 아니라면 타입을 만들자.

# 문자열은 열거 타입을 대신하기에 적절하지 않다 
- 상수를 열거할 때는 문자열보다 열거형이 낫다 
# 문자열은 혼합 타입을 대신하기에 적합하지 않다 
```java
String compoundKey = className + "#" + i.next();
```
- 구분을 위한 #가 두 요소에 쓰이면 혼란스러워진다
- 요소에 개별 접근하려면 문자열을 파싱해야 해서 느리다.
- 귀찮고 오류 가능성도 크다.
- String 제공 기능만 사용할 수 있다
- 적절한 클래스를 새로 만들자

# 문자열은 권한을 표현하기에 적합하지 않다. 

> ThreadLocal : 일반 변수는 특정 코드 블록 내에서만 유효하다. ThreadLocal을 사용하면 특정 스레드가 실행하는 모든 코드에서 그 스레드에 설정된 변수 값을 사용할 수 있다. 

```java
public class ThreadLocal {
    private ThreadLocal() { }

	// 현 스레드의 값을 키로 구분해 저장한다.
	public static void set(String key, Object value);
    
    // (키가 가리키는) 현 스레드의 값을 반환한다.
    public static Object get(String key);
}
```
- 권한(key)를 문자열로 관리한다
- 클라이언트가 중복되는 key를 사용하게 되면 문제가 발생한다.
- 악의적인 클라이언트로부터 보안에 취약하다.

```java
public class ThreadLocal {
    private ThreadLocal() {
    }
    
    public static class Key { // (권한)
        Key() { }
    }
    public staatic getKey() {
        return Key;
    }
    
    public static void set(Key key, Object value);
    public static Object get(Key key);
}
```
- 문자열 대신 Key 객체를 사용해 문제를 해결한다

```java
public final class ThreadLocal<T> {
    public ThreadLocal();
    public void set(T value);
    public T get();
}
```
- 약간 개선을 더해 이렇게 바꿀 수 있다.
- Key 객체를 생성할 수 있으니까 Key의 인스턴스 메서드로 set과 get을 넣으면 Key 자체가 스레드 지역변수가 된다.
- 제네릭으로 타입안전하게 만든다.
