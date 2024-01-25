- 스트림의 각 변환 단계는 가능한 한 순수함수여야 한다.
> 순수 함수: 입력만이 결과에 영향을 주는 함수, 다른 가변 상태를 참조하지 않고, 함수 스스로도 다른 상태를 변경하지 않는다

```java
        Map<String, Long> freq = new HashMap<>();
        try (Stream<String> words = new Scanner(file).tokens()) {
            words.forEach(word -> {
                freq.merge(word.toLowerCase(), 1L, Long::sum);
            });
        }
```
- forEach에서 외부 상태인 freq를 수정한다.
```java
        Map<String, Long> freq;
        try (Stream<String> words = new Scanner(file).tokens()) {
            freq = words
                    .collect(groupingBy(String::toLowerCase, counting()));
        }
```
- 수집기 colletor를 사용해 freq를 초기화한다.
- forEach에서는 스트림 계산 결과를 보고할 때만 사용하자

# collector
- 컬렉션의 원소들을 객체(일반적으로 컬렉션)  하나로 취급한다
- `toList()`, `toSet()`, `toCollection(collectionFactory)`가 각각 리스트, 집합, 컬렉션을 반환한다
```java
        List<String> topTen = freq.keySet().stream()
              .sorted(comparing(freq::get).reversed())
              .limit(10)
              .collect(toList());
```
- comparing은 비교자 생성 메서드다.
- 가장 빈도가 높은 단어 10개를 리스트로 만든다

---

다양한 collector 함수를 알아보자

### `toMap(keyMapper, valueMapper)`
스트림 원소를 키에 매핑하는 함수, 값에 매핑하는 함수를 인수로 받는다.
```java
        source.stream()
                .collect(Collectors.toMap(i -> i*10, i -> i+1));
```
- 스트림의 각 원소가 고유한 키에 매핑되어 있을 때 적합하다
```java
robots.stream().collect(Collectors.toMap(Robot::getCompany, robot -> robot, maxBy(comparing(Robot::getSales))));
```
- `binaryOperator<U>` 형태의 병합 함수를 전달할 수도 있다. U는 맵의 value 타입이며 같은 key를 공유하는 값들은 이 병합 함수를 통해 기존 값에 합쳐진다.
- key : 로봇 제조사, value : 로봇, 병합 함수 : 로봇의 판매량끼리 비교해서 더 많은 로봇 -> 즉 `<제조사,제조사별 가장 많이 팔린 로봇>` 의 맵을 만든다.
```java
robots.stream().collect(Collectors.toMap(Robot::getCompany, robot -> robot, (oldSales, newSales) -> newSales));
```
- 키에 대해 값이 여러개여서 충돌이 발생하면 마지막 값을 취하는 형태로도 병합 함수를 사용할 수 있다.
- 네번째 매개변수로 맵 팩터리를 받아 원하는 맵 구현체를 지정할 수 있다.
- toMap의 변종 toConcurrentMap은 병렬 실행된 후 결과로 ConcurrentHashMap 객체를 생성한다

### `groupingBy`
- 입력으로 분류함수를 받고 출력으로 원소들을 카테고리별ㄹ로 모아 놓은 맵을 담은 수집기를 반환한다
- 분류함수는 입력받은 원소가 속하는 카테고리를 반환한다. 카테고리가 맵 키로 쓰인다.
- 맵의 value는 key에 해당하는 모든 원소의 list다. 

```java
robots.stream().collect(groupingBy(Robot::getCompany));
```
- 분류 함수 하나를 받아 맵을 반환한다.

```java
        robots.stream().collect(groupingBy(Robot::getCompany, toSet()));

        robots.stream().collect(groupingBy(Robot::getCompany, counting()));
```
- 결과 map의 value를 list 말고 다른 형태를 원한다면 다운스트림 수집기를 전달한다.
- 다운스트림 수집기는 key의 모든 값을 담은 value 스트림으로부터 값을 생성한다. `toSet()`을 전달하면 Set을 생성한다.
- `counting()`을 전달해 원소의 개수를 value로 할 수 있다.
- 추가로 맵 팩터리를 전달해 맵의 타입을 지정할 수 있다.
- groupingByCouncurrent 메서드는 ConcurrenthashMap 인스턴스를 만든다.

### 다운스트림 수집기를 반환하는 Collections의 메서드 
- `counting()`
- summing, averaging, summarizing ~ int,long,double
- filtering, mapping ,flatMapping, collecting, AndThen

### `partitioningBy` 
- 분류 함수 자리에 Predicate를 받고 키가 Boolean인 맵을 반환한다.

### minBy, maxBy
- 인수로 받은 비교자를 이용해 스트림의 최소, 최대값 반환

### joining
- CharSequence 인스턴스의 스트림에만 적용할 수 있다
- 매개변수가 없는 joining : 원소 연결 `messironaldombappe`
- 매개변수 1개 joining : 구분문자를 받아서 사이에 넣는다 `messi,ronaldo,mbappe`
- 매개변수 3개 joining : prefix, 구분문자, suffix를 넣는다. `{messi,ronaldo,mbappe}`
