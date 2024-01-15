# 가변인수와 제네릭 
- 가변인수 메서드를 호출하면 가변인수를 담기 위한 배열이 자동으로 하나 만들어진다
> 실체화 불가 타입 : 런타임에 컴파일타임보다 타입 관련 정보를 적게 담고 있는 타입. 거의 모든 제네릭과 매개변수화 타입. 실체화 불가 타입으로 직접 배열을 선언할 수 없다

- 가변인수 메서드를 호출할 때도 varargs 매개변수가 실체화 불가 타입으로 추론되면 경고를 낸다.
- 매개변수화 타입(`List<String>`)의 변수가 타입이 다른 객체를 참조하면 힙 오염이 발생한다
- 매개변수화 타입이 다른 타입의 객체를 참조하면 컴파일러가 자동 형변환을 하는 상황이 발생할 수 있고, 형변환이 실패할 수 있다. 타입 안전성이 무너진다. 
```java
    static void dangerous(List<String>... stringLists) {
        List<Integer> intList = List.of(42);
        Object[] objects = stringLists;
        objects[0] = intList; // 힙 오염 발생
        String s = stringLists[0].get(0); // ClassCastException
    }
```
- 가변인수를 받는 메서드므로 가변인수를 담기 위한 배열(`List<String>`형 배열 stringLists)이 자동으로 만들어진다.
- `objects[0] = intList;`를 통해 `List<String>`이 `List<Integer>`를 가리키게 되었다. 매개변수화 타입(`List<String>`)의 변수가 타입이 다른 객체(`List<Integer>`)를 참조하므로 힙 오염이 발생한다
- `String s = stringLists[0].get(0);`에서 컴파일러가 자동 형변환을 하지만 Integer -> String이므로 ClassCastException을 던진다.

---

- 위에서 설명햇듯이 실체화 불가 타입인 제네릭으로 배열을 선언하는 것은 안전하지 않다. 따라서 프로그래머가 직접 생성하는 것은 허용하지 않는다. 
- 제네릭 가변인수를 받는 메서드는 실무에서 유용해 허용한다
  - `Arrays.asList(T... a)` , `Collections.addAll(Collection<? super T> c, T... elements)` 등이 이에 속한다. 이 메서드들은 타입 안전하다

# @SafeVarargs
- 자바7 전에는 메서드 호출자가 제네릭 가변인수 메서드를 호출할 때마다 @SuppressWarnings("unchecked") 애너테이션을 달아 경고를 숨겼다.
  - 가독성을 떨어트리고, 진짜 경고를 숨길수도 있다
- 자바 7부터 @SafeVarargs 애너테이션이 추가되어 메서드 작성자가 메서드가 타입 안전함을 보장하는 장치다. 애너테이션을 달면 컴파일러는 경고를 하지 않는다.
- 메서드가 안전한게 확실하지 않다면 절대 @SafeVarargs를 달지 말자. 제네릭이나 매개변수화 타입의 varargs 매개변수를 받는 모든 메서드에 @SafeVarargs를 달자. 즉 안전하지 않은 varargs 메서드를 작성하지 말자. 
  - 메서드가 안전하다는 것은
    1) 가변인수 매개변수 배열에 아무것도 저장하지 않는다
    2) 가변인수 매개변수 배열(혹은 복제본)을 신뢰할 수 없는 코드에 노출하지 않는다
      - 신뢰할 수 있는 코드란
        1) `@SafeVarargs`로 애노테이드 된 또 다른 varargs 메서드
        2) 배열 내용의 일부 함수를 호출만 하는 일반 메서드

가변인수 배열을 신뢰할 수 없는 코드에 노출하면 안되는 이유를 알아보자
```java
    static <T> T[] toArray(T... args) {
        return args;
    }
```
- 이 메서드는 자신의 가변인수 매개변수 배열에 아무것도 저장하지 않고 반환한다
- args 배열의 타입은 컴파일타임에 결정된다. 
```java
    static <T> T[] pickTwo(T a, T b, T c) {
        switch(ThreadLocalRandom.current().nextInt(3)) {
            case 0: return toArray(a, b);
            case 1: return toArray(a, c);
            case 2: return toArray(b, c);
        }
        throw new AssertionError(); // 도달할 수 없다.
    }
```
- pickTwo의 T 타입이 무엇이 되더라도 toArray의 가변인수 매개변수 배열은 담을수 있어야 하므로 `Object[]` 배열을 생성한다.
```java
    public static void main(String[] args) { // (194쪽)
        String[] attributes = pickTwo("좋은", "빠른", "저렴한");
        System.out.println(Arrays.toString(attributes));
    }
```
- pickTwo의 반환타입인 `Object[]`를 `String[]`로 자동 형변환 하는 과정에서 ClassCastException을 던진다.

> 따라서 제네릭 varargs 매개변수 배열에 다른 메서드가 접근하도록 허용하는 것은 안전하지 않다

# List 매개변수 
```java
    static <T> List<T> flatten(List<List<? extends T>> lists) {
        List<T> result = new ArrayList<>();
        for (List<? extends T> list : lists)
            result.addAll(list);
        return result;
    }

    public static void main(String[] args) {
        List<Integer> flatList = flatten(List.of(
                List.of(1, 2), List.of(3, 4, 5), List.of(6,7)));
        System.out.println(flatList);
    }
```
- 제네릭 varargs 매개변수를 List로 대체할 수 있다
```java
    @SafeVarargs
    @SuppressWarnings("varargs")
    static <E> List<E> of(E... elements) {
        switch (elements.length) { // implicit null check of elements
            case 0:
                @SuppressWarnings("unchecked")
                var list = (List<E>) ImmutableCollections.EMPTY_LIST;
                return list;
            case 1:
                return new ImmutableCollections.List12<>(elements[0]);
            case 2:
                return new ImmutableCollections.List12<>(elements[0], elements[1]);
            default:
                return ImmutableCollections.listFromArray(elements);
        }
    }
```
- `List.of`가 @SafeVarargs가 달린 제네릭 가변인수를 받는 메서드이므로 위와 같이 활용가능하다
- 직접 @SafeVarargs 애너테이션을 달지 않아도 된다
- 실수로 안전하다고 판단할 걱정도 없다
