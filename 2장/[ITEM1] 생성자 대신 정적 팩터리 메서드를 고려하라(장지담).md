정적 팩터리 메서드는 그 클래스의 인스턴스를 반환하는 단순한 정적 메서드다.

# 정적 팩터리 메서드의 장점

## 이름을 가질 수 있다
```java
public class Robot {
    private int battery;
    private int power;

    private Robot(int battery, int power) {
        this.battery = battery;
        this.power = power;
    }

    public static Robot of(int battery, int power) {
        return new Robot(battery, power);
    }
}
```
생성자는 클래스명으로 이름이 고정되는데 반해, 정적 팩터리 메서드는 이름을 가질 수 있다. 반환될 객체의 특성을 나타낼 수 있다.
예컨대 of는 여러 매개변수를 받아서 인스턴스를 반환한다는 의미를 나타낸다. 

## 호출될 떄마다 인스턴스를 새로 생성하지 않아도 된다
```
public class Robot {
    private int battery;
    private int power;

    private static Robot INSTANCE;

    private Robot(int battery, int power) {
        this.battery = battery;
        this.power = power;
    }

    public static Robot getInstance(int battery, int power) {
        if (INSTANCE == null) {
            INSTANCE = new Robot(battery, power);
        }
        return INSTANCE;
    }
}
```
생성자는 호출될 때 마다 객체를 생성해야 한다. 
정적 팩터리 메서드는 반복되는 요청에 같은 객체를 반환할 수 있다.
언제 어느 인스턴스를 살아있게 할지 통제하는 통제 클래스를 만들 수 있다.

## 반환 타입의 하위 타입 객체를 반환할 수 있다
```
public interface Test {
    public static Test getImpl() {
        return new TestImpl();
    }
}
```
반환형을 인터페이스로 하고, 구현 클래스는 공개하지 않고 반환해서 API를 작게 만들 수 있다. (구현체 API를 노출하지 않을 수 있다)
클라이언트는 구현체에 대해 알 필요가 없어 사용하기 편리하다. 
인터페이스를 정적 팩터리 메서드의 반환형으로 사용하는 것을 인터페이스 기반 프레임워크라 한다.

## 입력 매개변수에 따라 매번 다른 클래스의 객체를 반환할 수 있다
```
    public static <E extends Enum<E>> EnumSet<E> noneOf(Class<E> elementType) {
        Enum<?>[] universe = getUniverse(elementType);
        if (universe == null)
            throw new ClassCastException(elementType + " not an enum");

        if (universe.length <= 64)
            return new RegularEnumSet<>(elementType, universe);
        else
            return new JumboEnumSet<>(elementType, universe);
    }
```
EnumSet의 정적 팩터리 메서드 noneOf는 원소 개수에 따라 RegularEnumSet 또는 JumboEnumSet을 반환한다.
EnumSet의 하위 클래스 객체를 반환하기면 하면 되며, 클라이언트는 하위 클래스에 관해 알 필요가 없다.

## 정적 팩터리 메서드를 작성하는 시점에는 반환할 객체의 클래스가 존재하지 않아도 된다
서비스 제공자 프레임워크는 서비스의 구현체를 클라이언트에 제공하는 것을 프레임워크가 통제하여 클라이언트를 구현체로부터 분리하는 것이다.

- 서비스 제공자 프레임워크 구성요소
  - 서비스 인터페이스 : 구현체 동작 정의
  - 제공자 등록 API : 제공자가 구현체를 등록할 때 사용
  - 서비스 접근 API : 클라이언트가 서비스의 인스턴스를 얻을 때 사용
  - 서비스 제공자 인터페이스 : 구현체 동작을 정의하는 서비스 인터페이스의 객체를 생성하는 팩터리 객체를 설명
 
서비스 접근 API에서 정적 팩터리 메서드가 쓰인다. 
서비스 사용자 프레임워크인 JDBC를 살펴보자.
```
// jdbc 연결하는 코드
Class.forName("oracle.jdbc.driver.OracleDriver"); 
Connection conn = null; 
conn = DriverManager.getConnection("jdbc:oracle:thin:@localhost:1521:ORA92", "scott", "tiger"); 
Statement..
```
`DriverManager.getConnection`이 서비스 접근 API이다. 
```
public class DriverManager {
          	private DriverManager() {}
          	private static final Map<String,Driver> drivers = new ConcurrentHashMap<String,Driver>();
          	public static final String DEFAULT_DRIVER_NAME = "default";
          	public static void registerDefaultPrivider(Driver d) {
          		registerDriver(DEFAULT_DRIVER_NAME, d);
          	}
          	public static void registerDriver(String name, Driver d) {
          		drivers.put(name,d);
          	}
          	public static Connection getConnection() {
          		return getConnection(DEFAULT_DRIVER_NAME);
          	}
          	public static Connection getConnection(String name) {
          		Driver d = drivers.get(name);
          		if(d==null) 
          			throw new IllegalArgumentException();
          	 return d.getConnection();         
          	}          
          }
출처: https://devyongsik.tistory.com/294 [DEV용식:티스토리]
```
DriverManager의 정적 팩터리 메서드인 getConnection으로 서비스 인터페이스인 Connection을 반환한다. 
반환할 Connection은 제공자 등록 API인 registerDriver에 의해 등록된다.
따라서 정적 팩터리 메서드인 getConnection을 작성할 때는 반환할 객체의 클래스가 존재하지 않아도 된다. 
상황에 따라 적절한 Connection을 반환하게 정의해놓고 추후에 등록한 드라이버를 반환하면 된다. 

# 정적 팩터리 메서드의 단점
## 정적 팩터리 메서드만 제공하면 하위 클래스를 만들 수 없다
- 상속을 위해서는 public이나 protected 생성자가 필요하다
- 정적 팩터리 메서드만 제공하면 상속이 불가능하다.
  
하지만 이 단점은 상속보다 컴포지션(필드로 갖게 하는 것)을 사용하도록 유도하고, 불변 타입으로 만들려면 이 제약을 지커야 하므로 마냥 단점으로 볼 수는 없다. 

## 정적 팩터리 메서드는 프로그래머가 찾기 어렵다
- 생성자처럼 API 문서에 명확하게 드러나지 않는다
- API 문서를 잘 작성하고, 흔히 알려진 메서드 명명규칙을 지키는 것이 좋다

- 정적 팩터리 메서드 명명규칙
  - from : 매개변수 하나를 받아서 해당 타입의 인스턴스 반환
  - of : 여러 매개변수를 받아서 적합한 타입의 인스턴스 반환
  - valueOf : from, of의 자세한 버전
  - instance, getInstance : 매개변수를 받는다면 매개변수로 명시한 인스턴스를 반환한다. 하지만 같은 인스턴스임을 보장하지는 않는다.
  - create, newInstance : instance, getInstance와 같지만 매번 새로운 인스턴스를 반환한다.
  - getType : getInstance와 같지만, 생성할 클래스가 아닌 다른 클래스에 메서드를 정의할 때 쓴다. Type은 생성할 클래스 명이다.
    - `Car car = Factory.getCar();`
  - newType : newInstance와 같으나, 생성할 클래스가 아닌 다른 클래스에 메서드를 정의할 때 쓴다.
  - type : getType 또는 newType의 간결한 버전
    - `Car car = Factory.car();`
  













