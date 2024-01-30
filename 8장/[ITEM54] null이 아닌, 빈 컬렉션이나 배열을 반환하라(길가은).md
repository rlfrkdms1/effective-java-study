```java
public class CheeseFactory {

    private final List<Cheese> cheesesInStock = new ArrayList<>();

    public List<Cheese> getCheesesInStock() {
        return cheesesInStock.isEmpty() ? null : new ArrayList<>(cheesesInStock);
    }
}
```
위의 코드는 CheeseInStock이 비어있으면, 즉 저장되어있는 cheese가 없다면 null을 반환하고 있다면 List를 새로 생성해 저장되어 있는 치즈들을 반환하는 getter이다. 따라서 클라이언트는 이 getter를 사용할 때 null을 처리하는 코드를 추가로 작성해야한다. 

```java
        CheeseFactory cheeseFactory = new CheeseFactory();
        List<Cheese> cheeses = cheeseFactory.getCheesesInStock();
        if (cheeses != null && cheeses.contains(Cheese.STILTON)) {
            ...
        }
```
위의 코드에서 보듯이 `cheeses != null` 작성을 통해 추가로 null 체크를 해준것을 알 수 있다. 

클라이언트에서 이를 빼먹으면 cheeses가 null인 경우 NPE가 발생한다. 또한 null을 반환하려면 반환하는 쪽에서도 이 상황을 특별히 취급해줘야해 코드가 더 복잡해진다. 

# 빈 컨테이너 반환은 성능이 나빠지지 않나요?
하지만 null을 사용하지 않고 빈 컨테이너를 할당하는데는 비용이드니 그냥 null을 반환하는게 낫다고 생각할 수 있다. 이는 두가지 측면에서 틀린 주장이다. 

## 이 정도 성능차는 괜찮아요!
성능 분석 결과 이 할당이 성능 저하의 주범이라고 확인되지 않는 이 정도 성능 차이는 신경 쓸 수준이 못된다. 

## 새로 할당하지 않아도 반환할 수 있어요!

빈 컬렉션과 배열은 굳이 새로 할당하지 않고도 반환할 수 있다. 

```java
public class CheeseFactory {

    private final List<Cheese> cheesesInStock = new ArrayList<>();

    public List<Cheese> getCheesesInStock() {
        return new ArrayList<>(cheesesInStock);
    }

}
```
위와 같이 사용하면된다. 

가능성은 작으나, 사용패턴에 따라 빈 컬렉션 할당이 성능을 눈에 띄게 떨어뜨릴 수도 있다.

```java
    public List<Cheese> getCheesesInStock() {
        return cheesesInStock.isEmpty() ? Collections.emptyList() : new ArrayList<>(cheesesInStock);
    }
```
위와 같이 불변 컬렉션을 반환하는 `Collection.emptyList()`를 사용하면 된다. 

하지만 이 역시 최적화에 해당하므로 꼭 필요할 때만 사용하자. 

배열을 사용할 때도 마찬가지다. null이 아닌 길이가 0인 배열을 반환하는 것이 좋다. 

```java
    public Cheese[] getCheesesInStock() {
        return cheesesInStock.toArray(new Cheese[0]);
    }
```
`toArray`메서드는 아래와 같다. 즉, 인수로 들어온 배열의 크기 보다 collection의 size가 작은 경우 인수로 들어온 배열을 반환한다. 

```java
    public <T> T[] toArray(T[] a) {
        // Estimate size of array; be prepared to see more or fewer elements
        int size = size();
        T[] r = a.length >= size ? a :
                  (T[])java.lang.reflect.Array
                  .newInstance(a.getClass().getComponentType(), size);
        Iterator<E> it = iterator();

        for (int i = 0; i < r.length; i++) {
            if (! it.hasNext()) { // fewer elements than expected
                if (a == r) {
                    r[i] = null; // null-terminate
                } else if (a.length < i) {
                    return Arrays.copyOf(r, i);
                } else {
                    System.arraycopy(r, 0, a, 0, i);
                    if (a.length > i) {
                        a[i] = null;
                    }
                }
                return a;
            }
            r[i] = (T)it.next();
        }
        // more elements than expected
        return it.hasNext() ? finishToArray(r, it) : r;
    }
```
빈 배열을 계속 할당하는 것이 성능을 떨어뜨릴 것 같다면, 배열을 미리 선언해두고 계속 재사용해도 된다. 

```java
    private static final Cheese[] EMPTY_CHEESE_ARRAY = new Cheese[0];

    public Cheese[] getCheesesInStock() {
        return cheesesInStock.toArray(EMPTY_CHEESE_ARRAY);
    }
```
단순히 성능을 개선할 목적이라면 `toArray`에 넘기는 배열을 미리 할당하는 건 추천하지 않는다. 오히려 성능이 떨어진다는 연구 결과도 있다. 

```java
    public Cheese[] getCheesesInStock() {
        return cheesesInStock.toArray(new Cheese[cheesesInStock.size()]);
    }
```

#### 출처

이펙티브 자바 3/E
