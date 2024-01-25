# 스트림
스트림 API는 다량의 데이터 처리 작업을 돕고자 자바8에 추가되었다. 이 API가 제공하는 추상개념 중 핵심은 두가지이다. 

1. 스트림은 데이터 원소의 유한 혹은 무한 시퀀스를 뜻한다. 
2. 스트림 파이프라인은 이 원소들로 수행하는 연산 단계를 표현하는 개념이다. 

스트림 파이프라인은 소스 스트림에서 시작해 종단 연산으로 끝나며, 그 사이에 하나 이상의 중간 연산이 있을 수 있다. 

스트림 파이프라인은 지연 평가된다. 평가는 종단 연산이 호출될 때 이뤄지며, 종단 연산에 쓰이지 않는 데이터 원소는 계산에 쓰이지 않는다. 이러한 지연 평가가 무한 스트림을 다룰 수 있게 해주는 열쇠다. 

종단 연산을 빼먹는다면, 스트림 파이프라인은 아무 일도 하지 않는 명령어와 같으므로 빼먹지 말자.

스트림 API는 메서드 연쇄를 지원하는 플루언트 API이다. 즉, 파이프라인 하나를 구성하는 모든 호출을 연결하여 단 하나의 표현식으로 완성할 수 있다. 

스트림을 제대로 사용하면 프로그램이 짧고 깔끔해지지만, 잘못 사용하면 읽기 어렵고 유지보수도 힘들어진다. 

예시를 살펴보자. 이 프로그램은 사전 파일에서 단어를 읽어 사용자가 지정한 문턱값보다 원소 수가 많은 아나그램 그룹을 출력한다. 아나그램은 철자를 구성하는 알파벡이 같고 순서만 다른 단어를 말한다. 

맵을 통해 관리하는데, 키는 단어를 구성하는 철자들을 알파벳순으로 정렬한 값이다. 

```java
public class IterativeAnagrams {
    public static void main(String[] args) throws IOException {
        File dictionary = new File(args[0]);
        int minGroupSize = Integer.parseInt(args[1]);

        Map<String, Set<String>> groups = new HashMap<>();
        try (Scanner s = new Scanner(dictionary)) {
            while (s.hasNext()) {
                String word = s.next();
                groups.computeIfAbsent(alphabetize(word),
                        (unused) -> new TreeSet<>()).add(word);
            }
        }

        for (Set<String> group : groups.values())
            if (group.size() >= minGroupSize)
                System.out.println(group.size() + ": " + group);
    }

    private static String alphabetize(String s) {
        char[] a = s.toCharArray();
        Arrays.sort(a);
        return new String(a);
    }
}
```
`groups.computeIfAbsent(alphabetize(word), (unused) -> new TreeSet<>()).add(word);`를 사용하는데, `computeIfAbsent`는 맵 안에 키가 있는지 찾아 있다면 키에 매핑된 값을 반환하고 없다면 건네진 함수 객체를 키에 적용해 값을 계산, 매핑한 후 반환한다. 

그렇다면 스트림을 사용해보자. 아래의 코드는 무엇이 잘못된걸까?

```java
public class StreamAnagrams {
    public static void main(String[] args) throws IOException {
        Path dictionary = Paths.get(args[0]);
        int minGroupSize = Integer.parseInt(args[1]);

        try (Stream<String> words = Files.lines(dictionary)) {
            words.collect(
                    groupingBy(word -> word.chars().sorted()
                            .collect(StringBuilder::new,
                                    (sb, c) -> sb.append((char) c),
                                    StringBuilder::append).toString()))
                    .values().stream()
                    .filter(group -> group.size() >= minGroupSize)
                    .map(group -> group.size() + ": " + group)
                    .forEach(System.out::println);
        }
    }
}
```
스트림을 과하게 사용해 읽기 힘들다. 

> 스트림을 과용하면 프로그램이 읽거나 유지보수하기 어려워진다. 

그렇다면, 스트림을 적당히 사용한 프로그램의 모습은 어떨까?

```java
public class HybridAnagrams {
    public static void main(String[] args) throws IOException {
        Path dictionary = Paths.get(args[0]);
        int minGroupSize = Integer.parseInt(args[1]);

        try (Stream<String> words = Files.lines(dictionary)) {
            words.collect(groupingBy(word -> alphabetize(word)))
                    .values().stream()
                    .filter(group -> group.size() >= minGroupSize)
                    .forEach(g -> System.out.println(g.size() + ": " + g));
        }
    }

    private static String alphabetize(String s) {
        char[] a = s.toCharArray();
        Arrays.sort(a);
        return new String(a);
    }
}
```
원래 코드보다 짧아지고 명확해진 것을 알 수 있다. 코드가 이해하기 쉬워졌다. 

> 람다에서는 타입이름을 자주 생략하므로 매개변수 이름을 잘 지어야 스트림 파이프라인의 가독성이 유지된다. 
도우미 메서드를 적절히 활용하는 일의 중요성은 일반반복 코드에서보다 스트림 파이프라인에서 훨씬 크다. (타입 정보가 명시 X, 임시변수 자주 사용 때문)

`alphabetize` 메서드도 스트림으로 다르게 구현할 수 있지만, 명확성이 떨어지고 잘못 구현할 가능성이 커진다. 스트림은 char용 스트림을 지원하지 않기 때문이다. 

예시를 보자. 

```java
"Hello world!".chars().forEach(System.out::print);
```
위 코드는 Hello world!를 출력할 것 같지만 실제로는 721011081081113211911111410810033을 출력한다. `chars()`가 int 값을 반환하기 때문이다. 따라서 Hello world를 출력하고 싶다면, 출력시 형변환을 해줘야한다. 

```java
"Hello world!".chars().forEach(x -> System.out.print((char) x);
```
하지만 **char값들을 처리할 때는 스트림을 삼가는 편이 낫다. **

무작정 모든 코드를 스트림으로 바꾸라는 것이 아니라는 것을 알 수 있다. 따라서 **기존 코드는 스트림을 사용하도록 리팩터링 하되, 새 코드가 더 나아 보일 때만 반영하자.**

## 함수객체로는 할 수 없지만, 코드 블록으로는 할 수 있는 일

- 코드 블록에서는 범위 안의 지역변수를 읽고 수정할 수 있다. 하지만, 람다에서는 final이거나 사실상 final인 변수만 읽을 수 있고, 지역변수를 수정하는 것은 불가능하다. 
- 코드 블록에서는 return 문을 사용해 메서드에서 빠져나가거나, break나 continue문으로 블록 바깥의 반복문을 종료하거나 반복을 한 번 건너뛸 수 있다. 또한 메서드 선언에 명시된 검사 예외를 던질 수 있다. 하지만 람다로는 이 중 어떤 것도 할 수 없다. 

## 스트림과 안성맞춤인 일

- 원소들의 시퀀스를 일관되게 변환한다.
- 원소들의 시퀀스를 필터링한다.
- 원소들의 시퀀스를 하나의 연산을 사용해 결합한다.
- 원소들의 시퀀스를 컬렉션에 모은다
- 원소들의 시퀀스에서 특정 조건을 만족하는 원소를 찾는다.

## 스트림으로 처리하기 어려운 일

한 데이터가 파이프라인의 여러 단계를 통과할 때 이 데이터의 각 단계에서의 값들에 동시에 접근하기는 어려운 경우 
-> 스트림 파이프라인은 일단 한 값을 다른 값에 매핑하고 나면 원래의 값은 잃는 구조기 때문이다. 

메르센 소수를 출력하는 프로그램을 작성해보자. 메르센 수는 $$2^p-1$$ 형태의 수다. p가 소수이면 해당 메르센 수도 소수일 수 있는데, 이를 메르센 소수라 한다. 

```java
static Stream<BigInteger> primes() {
	return Stream.iterate(TWO, BigInteger::nextProbablePrime);
}
```
스트림 파이프라인의 가독성을 위해 원소의 정체를 알려주는 복수 명사로 메서드 이름을 정하자. 

```java
public class MersennePrimes {
    static Stream<BigInteger> primes() {
        return Stream.iterate(TWO, BigInteger::nextProbablePrime);
    }

    public static void main(String[] args) {
        primes().map(p -> TWO.pow(p.intValueExact()).subtract(ONE))
                .filter(mersenne -> mersenne.isProbablePrime(50))
                .limit(20)
                .forEach(System.out::println);
    }
}
```
최종적으로 위와같은 프로그램이 만들어질 것이다. 이제 우리가 각 메르센 소수의 앞에 지수 p를 출력하길 원한다고 해보자. 하지만 지수 p는 초기 스트림에서만 나타나므로 종단 연산에서는 접근할 수 없다. 

따라서 첫 번째 중간 연산에서 수행한 매핑을 거꾸로 수행해 지수를 알아내야 한다. 그렇다면 다음과 같은 종단 연산이 될 것이다. 

```java
.forEach(mp -> System.out.println(mp.bitLength() + ": " + mp));
```
지수는 단순히 숫자를 이진수로 표현한 다음 몇 비트인지를 세면 나온다. 

## 스트림 VS 반복

카드덱을 초기화하는 작업을 생각해보자. 이 작업은 두 집합의 원소들로 만들 수 있는 가능한 조합을 계산하는 문제로 데카르트 곱이라 부른다. 

### for-each

```java
public class Card {
    public enum Suit { SPADE, HEART, DIAMOND, CLUB }
    public enum Rank { ACE, DEUCE, THREE, FOUR, FIVE, SIX, SEVEN,
                       EIGHT, NINE, TEN, JACK, QUEEN, KING }

    private final Suit suit;
    private final Rank rank;

    @Override public String toString() {
        return rank + " of " + suit + "S";
    }

    public Card(Suit suit, Rank rank) {
        this.suit = suit;
        this.rank = rank;

    }
    private static final List<Card> NEW_DECK = newDeck();

    // 코드 45-4 데카르트 곱 계산을 반복 방식으로 구현 (275쪽)
    private static List<Card> newDeck() {
        List<Card> result = new ArrayList<>();
        for (Suit suit : Suit.values())
            for (Rank rank : Rank.values())
                result.add(new Card(suit, rank));
        return result;
    }

    public static void main(String[] args) {
        System.out.println(NEW_DECK);
    }
}
```
위의 코드는 for-each 반복문을 중첨해 구현한 코드이다. 아래는 스트림으로 구현한 코드이다. 

### Stream

```java
public class Card {
    public enum Suit { SPADE, HEART, DIAMOND, CLUB }
    public enum Rank { ACE, DEUCE, THREE, FOUR, FIVE, SIX, SEVEN,
                       EIGHT, NINE, TEN, JACK, QUEEN, KING }

    private final Suit suit;
    private final Rank rank;

    @Override public String toString() {
        return rank + " of " + suit + "S";
    }

    public Card(Suit suit, Rank rank) {
        this.suit = suit;
        this.rank = rank;

    }
    private static final List<Card> NEW_DECK = newDeck();

    private static List<Card> newDeck() {
        return Stream.of(Suit.values())
                .flatMap(suit ->
                        Stream.of(Rank.values())
                                .map(rank -> new Card(suit, rank)))
                .collect(toList());
    }

    public static void main(String[] args) {
        System.out.println(NEW_DECK);
    }
}
```
어떤게 더 좋아보이는가? 결국 개인 취향과 프로그래밍 환경의 문제다. 처음 방식은 더 단순하고, 더 자연스러워 보인다. 이해하고 유지보수하기에 처음 코드가 더 편한 프로그래머가 많겠지만, 두번째인 스트림 방식을 편하게 생각하는 프로그래머도 있다. 

나와, 내 코드를 보고 수정할 여지가 있는 동료들이 함께 이해하고 선호하는 방식을 사용하자. 

> 스트림과 반복 중 어느 쪽이 나은지 확신하기 어렵다면 둘 다 해보고 더 나은 쪽을 택하라.

#### 출처

이펙티브 자바 3/E


[이펙티브 자바 github](https://github.com/WegraLee/effective-java-3e-source-code/tree/master)

