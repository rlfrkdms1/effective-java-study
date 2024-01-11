같은 기능의 객체를 매번 생성하는 것 보다 객체 하나를 재사용하는 것이 좋다.

# String 

```java
        for (int i=0; i<10; i++){
            String s = new String("lionel messi");
            System.out.println(s);
        }
```
- 실행될 때 마다 String 객체를 생성한다.
- String의 인자로 전달하는 "lionel messi"와 생성 결과가 같다.
- 불필요한 String 객체를 9개 생성한다.
```java
        for (int i=0; i<10; i++){
            String s = "lionel messi";
            System.out.println(s);
        }
```
- 하나의 String을 재사용한다.
- 같은 문자열을 사용하는 모든 코드가 같은 객체를 재사용한다.

# 정적 팩터리 메서드
```java
public final class Robot {
    private static Robot INSTANCE;
    final int energy;

    public Robot getInstance() {
        if (INSTANCE == null) {
            INSTANCE = new Robot();
        }
        return INSTANCE;
    }

    private Robot() {
        energy = 10;
    }
}
```
- 불변 클래스에서 정적 팩터리 메서드를 제공하면 불필요한 객체 생성을 피할 수 있다.
- 가변 객체더라도 사용중에 변경되지 않음이 보장되면 재사용할 수 있다.

# 생성 비용이 비싼 객체 캐싱
```java
public class Robot {
    private static final ExpensiveArm ARM = new ExpensiveArm();
    
    static int punch(){
        return ARM.punch();
    }
}
```
- 생성 비용이 비싼 객체가 반복적으로 필요하다면 캐싱해서 재사용하자
> 캐싱 : static 멤버로 초기화해놓고 재사용 할 수 있다.
- ExpensiceArm의 생성 비용이 비싸기 때문에 캐싱해놓고 `punch()`를 호출할 때마다 재사용한다.
- ARM 필드를 초기화하고 나서 `punch()`가 한번도 호출되지 않는다면 ARM은 불필요하게 초기화 된 것이다. `punch()`가 처음 호출될 때 ARM을 초기화하는 지연 초기화 방식을 사용할 수 있다. 

# 어댑터
> 어댑터(뷰) : 구현체를 인터페이스에 맞게 감싸주는 중간 객체, 하나만 만들어서 재사용하면 된다. 

```java
    public Set<K> keySet() {
        Set<K> ks = keySet;
        if (ks == null) {
            ks = new KeySet();
            keySet = ks;
        }
        return ks;
    }
```
Map 인터페이스의 `keySet()`은 Map 객체의 키 전부를 담은 Set 뷰를 반환한다. 
위 코드는 HashMap 구현체의 keySet 메서드다. keySet 필드가 null일 경우 새로운 KeySet을 생성해 반환하지만, 아닐 경우 기존 keySet을 반환한다.
즉 객체를 재사용하고 있다.

```java
        Map<String, Integer> map = new HashMap<>();

        map.put("messi",10);
        map.put("son",7);

        // 어댑터
        Set<String> set1 = map.keySet();
        Set<String> set2 = map.keySet();

        set1.remove("messi");

        System.out.println(set2.size()); // 1
        System.out.println(map.size()); // 1
```
`keySet()`은 구현체인 HashMap을 인터페이스 Set에 맞게 감싸주는 객체, 즉 어댑터 Set을 반환한다. Set은 하나만 만들어서 재사용한다. 
set1, set2가 같은 Set을 가리키고, 그 Set은 구현체 map을 가리킨다. 
따라서 `set1.remove()`에 의해 set2의 원소도 제거되고 map의 원소도 제거된다. 

# 오토 박싱 
> 오토박싱 : 기본 타입과 박싱된 기본 타입을 섞어 쓸 때 자동 형변환을 해주는 것 
```java
        Long sum = 0L;
        for (long i = 0; i < 100; i++) {
            sum += i;
        }
```
- Long에 long을 더할 때마다 오토박싱에 의해 새로운 Long 객체가 만들어진다.
- 의도치 않은 오토박싱이 발생하지 않게 주의하자
- 가능하면 기본 타입을 사용하자

# 오해
- **객체 생성은 비싸니 피해야 한다는 의미가 아니다.**
- 고비용 객체가 아닌 경우 JVM의 가비지 컬렉터에게 맡기자.
- 새로운 객체를 만들어야 한다면 기존 객체를 재사용하지 마라.
    - 새로운 객체를 만들어야 하는 상황에서(방어적 복사가 필요한 상황에서) 객체를 재사용하면 버그와 보안 등 피해가 크다.


