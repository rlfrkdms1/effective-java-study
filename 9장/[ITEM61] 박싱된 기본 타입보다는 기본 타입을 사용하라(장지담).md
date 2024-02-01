# 기본 타입과 박싱된 기본 타입은 다르다

기본 타입 int -> 박싱된 기본 타입 Integer

1. 기본 타입은 값만 갖지만 박싱된 기본 타입은 값과 식별성을 갖는다.
```java
    public static void main(String[] args) {
        int a = 1;
        int b = 1;
        System.out.println(a==b);
        Integer aa = new Integer(1);
        Integer bb = new Integer(1);
        System.out.println(aa==bb); // false
        System.out.println(aa.equals(bb)); // true
    }
```
- aa와 bb는 값이 갖지만 다른 객체다.
- 박싱된 기본 타입에는 `==`가 아닌 `equals`를 사용하자 
2. 박싱된 기본 타입은 null을 가질 수 있다.
```java
public class Main { 
    static Integer i;
    public static void main(String[] args) {
        int num = 1;
        if (i==num){
            System.out.println("yoyo");
        }
    }
}
```
- `i==num`에서 NPE가 발생한다. Integer의 초기값이 null이기 때문이다.
- 박싱된 기본 타입과 기본 타입을 혼용한 연산에서 박싱된 기본 타입의 박싱이 자동으로 풀렸고, null을 언박싱하며 NPE가 발생한 것이다. 
3. 기본 타입이 박싱된 기본 타입에 비해 시간, 메모리 효율적이다. 
```java
    public static void main(String[] args) {
        Integer sum = 0;
        for (int i = 0; i <= Integer.MAX_VALUE; i++) {
            sum += 1;
        }
        System.out.println(sum);
    }
```
- sum이 Integer 타입이어서 박싱, 언박싱이 계속 일어나 매우 느리다.

> 박싱된 기본 타입과 기본 타입의 차이를 인지하지 않고 마구잡이로 사용하지 말자 

# 박싱된 기본 타입은 언제 사용할까?
- 타입 매개변수 (컬렉션 등) : 타입 매개변수로 기본 타입을 지원하지 않는다. `List<int>`는 불가능하다.
- 리플렉션을 통해 메서드를 호출할 때
