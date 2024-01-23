# 배열 VS 리스트

## 공변 VS 불공변
배열은 공변이고, 제네릭은 불공변이다. 

|공변|불공변|
|---|----|
|String 이 Object의 서브타입이면, `List<String>`은 `List<? extend Object>` 의 서브타입이다.|List<String>은 List<Object>의 하위타입이 아니다. |

```java
Object[] objects = new Long[1];
objects[0] = "못 넣는다.";
```
위와 같이 Long용 저장소에 String인 문자열을 넣으면 컴파일에러가 터지지 않고 `ArrayStoreException` 런타임에러가 난다. 

그렇다면 리스트는 어떨까? 
```java
List<Object> objects = new ArrayList<Long>();
objects.add("string");
```
위와 같이 배열과 같은 형식으로 선언하고 원소를 넣어줬다. 그러면 `List<Object> objects = new ArrayList<Long>();`에서 아래와 같은 컴파일에러가 나는 것이다. 

```
Incompatible types. 
Found: 'java.util.ArrayList<java.lang.Long>', 
required: 'java.util.List<java.lang.Object>'
```

그렇다면, 런타임 에러와 컴파일 에러 중 뭐가 더 좋은가? 당연히 컴파일 에러로 알아차리는 것이 훨씬 좋다. 

## 배열은 실체화된다. 

배열은 런타임에도 자신이 담기로 한 원소의 타입을 인지하고 확인한다. 따라서 위에서 런타임 에러가 발생한 것이다. 하지만, 제네릭은 타입 정보가 런타임에는 소거된다. 원소타입을 **컴파일 타임**에만 검사하며 런타임에는 알 수 없다는 것이다. 

```java
/**
 * Thrown to indicate that an attempt has been made to store the
 * wrong type of object into an array of objects. For example, the
 * following code generates an <code>ArrayStoreException</code>:
 * <blockquote><pre>
 *     Object x[] = new String[3];
 *     x[0] = new Integer(0);
 * </pre></blockquote>
 *
 * @author  unascribed
 * @since   1.0
 */
public
class ArrayStoreException extends RuntimeException
```

따라서 배열은 제네릭 타입, 매개변수화 타입, 타입 매개변수로 사용할 수 없다. 

## 제네릭 배열을 막은 이유?
> 타입 안전하지 않기 때문이다. 

이를 허용하면 런타임에 `ClassCastException`이 발생할 수 있다. 런타임에 `ClassCastException`이 발생하는 것을 막아주겠다는 제네릭 타입 시스템의 취지에 어긋나는 것이다.  

```java
// 컴파일 전 (.java)
List<Integer> dice = List.of(1,2,3,4,5,6);
List<Integer> dices = new ArrayList<>();

// 컴파일 후 (.class)
List localList = List.of(Integer.valueOf(1), Integer.valueOf(2), Integer.valueOf(3), Integer.valueOf(4), Integer.valueOf(5), Integer.valueOf(6));
ArrayList localArrayList = new ArrayList();
```
제네릭은 런타임에 소거되어 타입을 알 수 없다. 위 처럼 컴파일 후엔 Integer라는 것을 알 수 없다. 

```java
class Example {
    public static void main(String[] args) {
        List<String>[] stringLists = new List<String>[1];
        List<Integer> intList = List.of(42);
        Object[] objects = stringLists;
        objects[0] = intList;
        String s = stringLists[0].get(0);
    }
}
```
실제로 위의 예시를 적용하면, 첫줄에서 컴파일에러가 난다. 그러므로 첫줄의 `List<String>[] stringLists = new List<String>[1];`가 허용된다고 가정해보자. 

```java
List<Integer> intList = List.of(42);
```
원소가 42 하나인 Integer 리스트가 생성된다. 

```java
Object[] objects = stringLists;
```
첫번째 줄에서 생성했던 `List<String>[]`인 stringLists를 `Object` 배열에 할당한다. 배열은 공변이므로 문제가 생기지 않는다. 

```java
objects[0] = intList;
```
`stringLists[0]` 에는 intList 인스턴스가 저장되어 있고, get 메서드를 통해 조회 및 자동 형변환 시 ClassCastException 발생하는 것이다. 

배열로 형변환할 때 제네릭 배열 생성 오류나 비검사 형변환 경고가 뜨는 경우 대부분은 배열인 `E[]` 대신 컬렉션인 `List<E>`를 사용하면 해결된다.  코드가 조금 복잡해지고 성능이 살짝 나빠질 수도 있지만, 그 대신 타입 안전성과 상호운용성은 좋아진다. 

```java
public class Chooser {
    private final Object[] choiceArray;

    public Chooser(Collection choices) {
        choiceArray = choices.toArray();
    }

    public Object choose() {
        Random rnd = ThreadLocalRandom.current();
        return choiceArray[rnd.nextInt(choiceArray.length)];
    }

}
```
위 클래스가 있다고 생각해보자. Chooser는 컬렉션 원소 중 하나를 무작위로 선택해 반환하는 choose 메서드를 제공한다. 이를 사용하려면 choose 메서드를 호출할 때마다 반환된 Object를 원하는 타입으로 형변환해야한다. 혹시나 다른 타입이 들어있었다면, 런타임에 형변환 오류가 날 것이다. 

```java
    public static void main(String[] args) {
        List<Object> integerList = List.of(1, "2", 3);

        Chooser chooser = new Chooser(integerList);

        for (int i = 0; i < 10; i++) {
            Number choice = (Number) chooser.choose();
            String choice = (String) chooser.choose();
        }
    }
```
위에서 `choose()`를 `String`으로 형변환 했을 때 `ClassCastException`이 발생했다. 따라서 위 클래스를 제네릭으로 바꿔보자. 

```java

public class Chooser<T> {
    private final T[] choiceArray;

    public Chooser(Collection<T> choices) {
        choiceArray = (T[]) choices.toArray();
    }

    public Object choose() {
        Random rnd = ThreadLocalRandom.current();
        return choiceArray[rnd.nextInt(choiceArray.length)];
    }
}
```
하지만 T가 무슨 타입인지 알 수 없으니 컴파일러는 이 형변환이 런타임에도 안전한지 보장할 수 없다는 메시지다. 제네릭에서는 원소의 타입 정보가 소거되어 런타임에는 무슨 타입인지 알 수 없음을 기억하자. 

위 프로그램은 동작하지만, 컴파일러가 안전을 보장하지 못한다. 따라서 비검사 형변화 경고를 제거하려면 배열대신 리스트를 사용하면 된다. 
```java
public class Chooser<T> {
    private final List<T> choiceList;

    public Chooser(Collection<T> choices) {
        choiceList = new ArrayList<>(choices);
    }

    public T choose() {
        Random rnd = ThreadLocalRandom.current();
        return choiceList.get(rnd.nextInt(choiceList.size()));
    }

    public static void main(String[] args) {
        List<Integer> intList = List.of(1, 2, 3, 4, 5, 6);

        Chooser<Integer> chooser = new Chooser<>(intList);

        for (int i = 0; i < 10; i++) {
            Number choice = chooser.choose();
            System.out.println(choice);
        }
    }
}
```
위와 같이 변경할 수 있는데, 이는 List를 Integer로 선언해 중간에 string이 들어가면 컴파일 에러가 발생하게 해주고, 혹여나 `List<Object>`로 안에 String이 섞여 들어갔다고 해도 `Chooser<Integer> chooser = new Chooser<>(intList);`에서 컴파일 에러가 난다. 



#### 출처

이펙티브 자바 3/E
[실체화](https://tlatmsrud.tistory.com/141)
[공변과 불공변](https://kdhyo98.tistory.com/83)
