```java
        int[] a = new int[10];
        //       1번  2번          3번
        for (int i = 0; i < a.length; i++) {
            //  4번
            // a[i]
        }

        List<Integer> list = new ArrayList<>();
        for (Iterator<Integer> it = list.iterator(); it.hasNext(); ) {
            Integer i = it.next();
        }
```
- 컬렉션을 반복자로 순회하거나, 배열을 인덱스로 순회하는 코드
- 지저분하다. 
- 부수적인 코드인 반복자, 인덱스가 자주 등장해 실수 확률을 높인다. 실제 필요한 것은 원소(int나 Integer)다. 
- 컬렉션인지 배열인지에 따라 코드 형태가 달라진다. 

# for-each문을 사용하자
for each문은 향상된 for문이다.
```java
        for (Integer e : a) {

        }

        for (Integer e : list) {
            
        }
```
- 컬렉션이던 배열이던 속도는 그대로다 

### 중첩 순회
```java
        for (Iterator<Suit> i = suits.iterator(); i.hasNext(); )
            for (Iterator<Rank> j = ranks.iterator(); j.hasNext(); )
                deck.add(new Card(i.next(), j.next()));
```
- `i.next()`는 내부 반복문 밖에서 호출해야 한다.
- 반복문 작성 시 이런 실수는 흔하다. 디버깅하기도 어렵다.
- 경우에 따라 예외를 던지지 않고 이상한 결과를 얻을 수 있다. 
```java
        for (Iterator<Suit> i = suits.iterator(); i.hasNext(); ) {
            Suit suit = i.next();
            for (Iterator<Rank> j = ranks.iterator(); j.hasNext(); )
                deck.add(new Card(suit, j.next()));
        }
```
- 위와 같이 바꿔줘야 한다. 

```java
                for (Suit suit : suits)
                    for (Rank rank : ranks)
                        deck.add(new Card(suit, rank));
```
- for-each를 사용하면 훨씬 간결하다.

# for-each를 사용할 수 없는 경우 
### 파괴적인 필터링 
- 컬렉션을 순회하면서 선택된 원소를 제거하려면 반복자의 remove를 호출해야 한다.
```java
        List<Integer> list = new ArrayList<>();
        for (Iterator<Integer> it = list.iterator(); it.hasNext(); ) {
            Integer i = it.next();
            if (i % 3 == 0) {
                it.remove();
            }
        }

        //removeIf 사용 : 명시적 순회가 필요없다 
        list.removeIf(i -> i % 3 == 0);

```
- java8부터 Collection의 removeIf를 사용하면 컬렉션을 명시적으로 순회할 필요가 없다.

    
### 변형
```java
        for (Robot robot : list) {
            if (robot.getPower() == 1) {
                // robot.setPower(10);
                robot = new Robot(10);
            }
        }
        System.out.println(list); //원소가 교체되지 않음!

        for (int i = 0; i < list.size(); i++) {
            if (list.get(i).getPower() == 1) {
                list.set(i, new Robot(10));
            }
        }
        System.out.println(list);
```
- forEach에서는 원소를 교체할 수 없다.
- 반복자나 인덱스를 사용해야 원소를 교체할 수 있다. 

### 병렬 반복 
- 여러 컬렉션을 병렬로 순회한다면 각각의 반복자, 인덱스 변수를 사용한다.
```java
        for (Iterator<Robot> it1 = list1.iterator(), it2 = list2.iterator(); it1.hasNext() && it2.hasNext();) {
            
        }
```

---

> Iterable을 구현한 모든 클래스에 for-each를 적용할 수 있다. 
