> 적합한 인터페이스만 있다면 매개변수뿐 아니라 반환값, 변수, 필드를 전부 인터페이스 타입으로 선언하라. 

객체의 실제 클래스를 사용해야할 상황은 오직 생성자로 생성할 때 뿐이다. 

```java
Set<Son> sonSet = new LinkedHashSet<>();
```
위와 같이 실제 클래스는 `LinkedHashSet`이지만 인터페이스 타입인 `Set`으로 선언한 것을 볼 수 있다. 

아래와 같이 클래스 타입으로 선언하는것은 안좋은 예시이다. 
```java
LinkedHashSet<Son> sonSet = new LinkedHashSet<>();
```

> 인터페이스를 타입으로 사용하는 습관을 길러두면 프로그램이 훨씬 유연해질 것이다. 

만약 위에서 사용했던 `LinkedHashSet`이 아닌 `HashSet`으로 변경하고 싶다면 다음과 같이 바꿔주면 된다. 

```java
Set<Son> sonSet = new HashSet<>();
```

클라이언트는 기존 클래스인 `LinkedHashSet`의 존재를 몰랐으므로 `HashSet`으로 변경해도 아무 문제가 없다. 

하지만 이때 주의해야할 점은 원래의 클래스가 인터페이스의 일반 규약 이외의 특별한 기능을 제공하며, 주변 코드가 이 기능에 기대어 동작한다면 새로운 클래스도 반드시 같은 기능을 제공해야한다. 

예로 위의 `LinkedHashSet`은 삽입한 순서를 보장해주지만 `HashSet`은 이를 보장해주지 않는다. 아래를 보자. 

```java
    public static void main(String[] args) {
        Set<String> linkedHashSet = new LinkedHashSet<>();
        Set<String> hashSet = new HashSet<>();
        
        List<String> target = List.of("effective Java", " ", "Chapter9 ", " ", "item64 ", " ", "객체는 인터페이스를 사용해 참조하라");
        
        linkedHashSet.addAll(target);
        hashSet.addAll(target);

        System.out.println("linkedHashSet = " + linkedHashSet);
        System.out.println("hashSet = " + hashSet);
    }
```
위의 출력은 아래와 같다. 

```
linkedHashSet = [effective Java,  , Chapter9 , item64 , 객체는 인터페이스를 사용해 참조하라]
hashSet = [ , effective Java, Chapter9 , 객체는 인터페이스를 사용해 참조하라, item64 ]
```
`linkedHashSet`을 사용했을 땐 삽입 순서대로 출력되었지만, `hashSet`은 그렇지 않다. 따라서 `LinkedHashSet`을 사용하던 코드가 이러한 순서정책에 의존해 작성되었다면 `HashSet`으로 마음대로 교체해서는 안된다. 

그렇다면 왜 구현타입을 바꾸려고 하는 것일까?
> 원래 것보다 성능이 좋거나, 멋진 신기능을 제공하기 때문일 수 있다. 

예를 들어 열거타입을 key로 갖는 HashMap을 참조하던 변수를 EnumMap을 참조하도록 바꾸면 속도가 빨라지고 순회 순서도 키의 순서와 같아진다. 

```java
    enum Season {SPRING, SUMMER, FALL, WINTER; }

        Map<Season, Integer> hashMap = new HashMap<>();
        Map<Season, Integer> enumMap = new EnumMap<>(Season.class);

```
선언 타입과 구현 타입을 동시에 바꿀 수 있으니 변수를 구현 타입으로 선언해도 괜찮을 거라 생각할 수 있다. 하지만 클라이언트가 기존 타입에만 있는 메서드나, 기존 타입을 입력으로 받는 메서드를 사용했을 경우엔 컴파일 되지 않는다.

따라서 인터페이스 타입으로 변수를 선언하면 이런 일이 발생하지 않는다. 

> 적합한 인터페이스가 없다면 당연히 클래스로 참조해야한다. 

- String과 BigInteger같은 값 클래스
- 클래스 기반으로 작성된 프레임 워크가 제공하는 객체들
	- OutputStream등 java.io패키지의 여러 클래스
- 인터페이스에는 없는 특별한 메서드를 제공하는 클래스 
 	- PriorityQueue는 Queue인터페이스에 없는 comparator 메서드를 제공한다. 
    
클래스 타입을 직접 사용하는 경우는 최소화해야하고, 절대 남발하지 말아야한다. 

> 적합한 인터페이스가 없다면 클래스의 계층구조 중 필요한 기능을 만족하는 가장 덜 구체적인(상위의) 클래스를 타입으로 사용하자. 


#### 출처

이펙티브 자바 3/E
