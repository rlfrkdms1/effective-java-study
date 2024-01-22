# ordinal을 사용한 배열

배열이나 리스트에서 원소를 꺼낼 때 ordinal 메서드로 인덱스를 얻는 코드가 있다. 아래를 살펴보자. 

```java
class Plant {
    enum LifeCycle { ANNUAL, PERENNIAL, BIENNIAL }

    final String name;
    final LifeCycle lifeCycle;

    Plant(String name, LifeCycle lifeCycle) {
        this.name = name;
        this.lifeCycle = lifeCycle;
    }

    @Override public String toString() {
        return name;
    }
```
이때 식물의 생명주기 별로 배열을 만들어 식물들을 관리하되, 상수의 순서별로 배열의 index를 결정한다고 해보자. 그렇다면 아래와 같은 코드가 만들어질 것이다. 

```java
    public static void main(String[] args) {
        Plant[] garden = {
            new Plant("바질",    LifeCycle.ANNUAL),
            new Plant("캐러웨이", LifeCycle.BIENNIAL),
            new Plant("딜",      LifeCycle.ANNUAL),
            new Plant("라벤더",   LifeCycle.PERENNIAL),
            new Plant("파슬리",   LifeCycle.BIENNIAL),
            new Plant("로즈마리", LifeCycle.PERENNIAL)
        };

        Set<Plant>[] plantsByLifeCycleArr =
                (Set<Plant>[]) new Set[Plant.LifeCycle.values().length];
        for (int i = 0; i < plantsByLifeCycleArr.length; i++)
            plantsByLifeCycleArr[i] = new HashSet<>();
        for (Plant p : garden)
            plantsByLifeCycleArr[p.lifeCycle.ordinal()].add(p);
        // 결과 출력
        for (int i = 0; i < plantsByLifeCycleArr.length; i++) {
            System.out.printf("%s: %s%n",
                    Plant.LifeCycle.values()[i], plantsByLifeCycleArr[i]);
        }
```
동작은 하지만, 배열의 index를 상수의 위치로 결정하는 것은 해선 안되며 배열을 사용하는 것은 좋지 않다. 제네릭과 호환되지 않으니 비검사 형변환을 수행해야하고, 깔끔히 컴파일 되지 않는다. 또한 상수의 위치가 바뀐다면 문제가 커진다. 

# EnumMap
이에대한 대안으로 EnumMap이 있다. 

```java
        Map<Plant.LifeCycle, Set<Plant>> plantsByLifeCycle =
                new EnumMap<>(Plant.LifeCycle.class);
        for (Plant.LifeCycle lc : Plant.LifeCycle.values())
            plantsByLifeCycle.put(lc, new HashSet<>());
        for (Plant p : garden)
            plantsByLifeCycle.get(p.lifeCycle).add(p);
        System.out.println(plantsByLifeCycle);
```
위와 같이 변경할 수 있는데, 더 짧고 명료하고 안전하고 성능도 원래 버전과 비등하다. 안전하지 않은 형변환은 쓰지 않고, 맵의 key인 열거 타입이 그 자체로 출력용 문자열을 제공하니 출력 결과에 직접 레이블을 달지 않아도 된다. 또한 index가 잘못될 일이 없다. 

`EnumMap`의 성능이 이전과 비견되는 이유는 내부에서 배열을 사용하기 때문이다. `EnumMap` 내부를 살펴보면 아래와 같다. 

```java
public class EnumMap<K extends Enum<K>, V> extends AbstractMap<K, V>
    implements java.io.Serializable, Cloneable
{
    /**
     * The {@code Class} object for the enum type of all the keys of this map.
     *
     * @serial
     */
    private final Class<K> keyType;

    /**
     * All of the values comprising K.  (Cached for performance.)
     */
    private transient K[] keyUniverse;

    /**
     * Array representation of this map.  The ith element is the value
     * to which universe[i] is currently mapped, or null if it isn't
     * mapped to anything, or NULL if it's mapped to null.
     */
    private transient Object[] vals;
	...
```
EnumMap의 생성자가 받는 키 타입의 Class 객체는 한정적 타입 토큰으로, 런타임 제네릭 타입 정보를 제공한다. 

## Stream 사용
스트림을 사용하면 코드를 더 줄일 수 있다. 아래를 보자. 

```java
        System.out.println(Arrays.stream(garden)
                .collect(groupingBy(p -> p.lifeCycle)));
```
하지만 이는 EnumMap을 사용하지 않아 EnumMap이 제공하는 이점이 사라진다. 
따라서 EnumMap을 사용하도록 바꿔보자. `groupingBy`는 mapFactory에서 매개변수에 원하는 맵 구현체를 명시해 호출할 수 있다. 따라서 EnumMap을 명시해 호출하면 된다. 

```java
        System.out.println(Arrays.stream(garden)
                .collect(groupingBy(p -> p.lifeCycle,
                        () -> new EnumMap<>(LifeCycle.class), toSet())));
```
스트림을 사용하면 EnumMap만 사용했을 때와는 다르게 동작한다. 

- EnumMap : 생애주기당 하나씩의 중첩 맵 생성
- 스트림 : 해당 생애주기에 속하는 식물이 있을 때만 생성

=> ANNUAL, PERENNIAL, BIENNIAL중 ANNUAL, PERENNIAL는 있고 BIENNIAL는 존재하지 않는다면 EnumMap에서는 3개, 스트림에선 두개를 생성하는 것이다. 

## 두 열거 타입 매핑

두 가지 상태(Phase)를 전이(Transition)와 매핑하도록 구현한 프로그램을 살펴보자. 

ex. LIQUID에서 SOLID로의 전이는 FREEZE가 되고, LIQUID에서 GAS로의 전이는 BOIL이 된다. 

```java
public enum Phase {
    SOLID,
    LIQUID,
    GAS;

    public enum Transition {
        MELT,
        FREEZE,
        BOIL,
        CONDENSE,
        SUBLIME,
        DEPOSIT;

        private static final Transition[][] TRANSITIONS = {
                {null, MELT, SUBLIME},
                {FREEZE, null, BOIL},
                {DEPOSIT, CONDENSE, null}
        };

        public static Transition from(Phase from, Phase to) {
            return TRANSITIONS[from.ordinal()][to.ordinal()];
        }
    }
}
```
괜찮은 것 같지만 절대 이렇게 사용해선 안된다. 컴파일러는 ordinal과 배열 인덱스의 관계를 알 도리가 없다. 즉, 열거 타입을 수정하면서 TRANSITIONS를 수정하지 않거나 잘못수정하면 런타임 오류가 날 것이다. 또한 상수가 늘어날 수록 null로 채워지는 칸도 많아진다. 따라서 EnumMap을 사용하자. 

```java
public enum Phase {
    SOLID, LIQUID, GAS;
    public enum Transition {
        MELT(SOLID, LIQUID), FREEZE(LIQUID, SOLID),
        BOIL(LIQUID, GAS), CONDENSE(GAS, LIQUID),
        SUBLIME(SOLID, GAS), DEPOSIT(GAS, SOLID);

        private final Phase from;
        private final Phase to;
        Transition(Phase from, Phase to) {
            this.from = from;
            this.to = to;
        }

        // 상전이 맵을 초기화한다.
        private static final Map<Phase, Map<Phase, Transition>>
                m = Stream.of(values()).collect(groupingBy(t -> t.from,
                () -> new EnumMap<>(Phase.class),
                toMap(t -> t.to, t -> t,
                        (x, y) -> y, () -> new EnumMap<>(Phase.class))));
        
        public static Transition from(Phase from, Phase to) {
            return m.get(from).get(to);
        }
    }

    // 간단한 데모 프로그램 - 깔끔하지 못한 표를 출력한다.
    public static void main(String[] args) {
        for (Phase src : Phase.values()) {
            for (Phase dst : Phase.values()) {
                Transition transition = Transition.from(src, dst);
                if (transition != null)
                    System.out.printf("%s에서 %s로 : %s %n", src, dst, transition);
            }
        }
    }
}
```
맵의 타입이 Map<Phase, Map<Phase, Transition>>인 것을 볼 수 있는데, 이를 위해 첫번째로 groupingBy, 두번째로 toMap을 사용했다. 이때 toMap의 (x, y) -> y는 선언만 하고 쓰이지 않는데, 이는 단지 EnumMap을 얻으려면 맵 팩터리가 필요하고 수집기들은 점층적 팩터리를 제공하기 때문이다. 

여기에 새로운 상태 PLASMA를 추가해보자. 이를 추가하면 연결되는 전이는 두개이다. 

GAS -> PLASMA : IONIZE
PLASMA -> GAS : DEIOINIZE

배열로 만든 코드를 수정하려면 상수를 각각 1,2개를 추가하고 배열을 16개짜리로 교체해야한다. 하지만 위에선 상태목록에 PLASMA를 추가하고 전이목록만 추가하면 된다. 

```java
public enum Phase {
    SOLID, LIQUID, GAS, PLASMA;
    public enum Transition {
        MELT(SOLID, LIQUID), FREEZE(LIQUID, SOLID),
        BOIL(LIQUID, GAS), CONDENSE(GAS, LIQUID),
        SUBLIME(SOLID, GAS), DEPOSIT(GAS, SOLID),         	  	  IONIZE(GAS, PLASMA), DEIONIZE(PLASMA, GAS);

        private final Phase from;
        private final Phase to;
        Transition(Phase from, Phase to) {
            this.from = from;
            this.to = to;
        }

        // 상전이 맵을 초기화한다.
        private static final Map<Phase, Map<Phase, Transition>>
                m = Stream.of(values()).collect(groupingBy(t -> t.from,
                () -> new EnumMap<>(Phase.class),
                toMap(t -> t.to, t -> t,
                        (x, y) -> y, () -> new EnumMap<>(Phase.class))));
        
        public static Transition from(Phase from, Phase to) {
            return m.get(from).get(to);
        }
    }

    // 간단한 데모 프로그램 - 깔끔하지 못한 표를 출력한다.
    public static void main(String[] args) {
        for (Phase src : Phase.values()) {
            for (Phase dst : Phase.values()) {
                Transition transition = Transition.from(src, dst);
                if (transition != null)
                    System.out.printf("%s에서 %s로 : %s %n", src, dst, transition);
            }
        }
    }
}
```

이를 통해 EnumMap을 사용하면 유지보수가 더 간단함을 알 수 있다. 
