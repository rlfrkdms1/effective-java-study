- 매개변수화 타입은 불공변이다. A가 B의 상위클래스더라도, `List<A>`와 `List<B>`는 관계가 없다.
```java
public class Hospital<T> {
    List<T> patients;

    public void addAll(List<T> list) {
        patients.addAll(list);
    }

    public void popAll(List<T> list) {
        for (int i = 0; i < patients.size(); i++) {
            list.add(patients.get(i));
        }
    }
}
```
```java
    public static void main(String[] args) {
        Hospital<Animal> hospital = new Hospital<>();
        List<Dog> dogs = new ArrayList<>();
        dogs.add(new Dog(1));
        dogs.add(new Dog(2));
        hospital.addAll(dogs); // 컴파일오류 발생
    }
```
- `Hospital<Animal>`에 `addAll(List<Dog>)`를 호출할 수 있을까? (Dog이 Animal을 상속한다)
- Dog가 Animal을 상속한다고 해서 `List<Dog>`가 `List<Animal>`을 상속하는 것이 아니기 때문에 호출할 수 없다.

  ```java
      public static void main(String[] args) {
        Hospital<Animal> hospital = new Hospital<>();
        List<Object> list = new ArrayList<>();
        hospital.popAll(list); // 컴파일오류 발생
    }
  ```
- 역시 Object는 Animal의 상위클래스이지만, `List<Object>`는 `List<Animal>`의 상위클래스가 아니므로 `popAll(List<Object>)`를 할 수 없다. 

# 한정적 와일드카드
- 제네릭은 불공변이지만 위의 예시들처럼 상속관계에 따라 논리적으로 동작하면 좋을 것 같은 상황이 있다. 이럴 때 한정적 와일드카드를 사용하자.
```java
public class Hospital<T> {
    List<T> patients;

    public void addAll(List<? extends T> list) {
        patients.addAll(list);
    }

    public void popAll(List<? super T> list) {
        for (int i = 0; i < patients.size(); i++) {
            list.add(patients.get(i));
        }
    }
}
```
- `List<? extends T> list>`는 List의 타입 매개변수가 T의 하위타입이면 된다는 뜻이다.
- `List<? super T> list`는 List의 타입 매개변수가 T의 상위타입이면 된다는 뜻이다.
- 원소의 생산자나 소비자용 입력 매개변수에 와일드카드 타입을 사용하자
- 입력 원소가 생산자, 소비자 역할을 동시에 한다면 타입을 정확히 지정해야 하므로 와일드카드를 사용하지 말자

## PECS

> PECS : producer extends, customer super

- 매개변수가 T의 생산자라면 `<? extends T>` 사용
  - addAll에서 list는 T를 생산한다
- 매개변수가 T의 소비자라면 `<? super T>` 사용
  - addAll에서 list는 T를 소비한다

> 반환 타입에는 한정적 와일드카드 타입을 사용하면 안된다. 클라이언트 코드에서도 와일드카드 타입을 써야 하기 때문이다. 클래스 사용자가 와일드카드 타입을 신경써야 한다면 API에 문제가 있을 가능성이 크다.
```java
public class Hospital<T> {

    public List<? extends Animal> ret1() {
        List<Dog> dogs = new ArrayList<>();
        return dogs;
    }

    public List<? super Animal> ret2() {
        List<Object> objects = new ArrayList<>();
        return objects;
    }
}
```
반환형이 한정적 와일드카드 타입인 메서드를 사용하면 안되는 이유를 알아보고자 반환형이 한정적 와일드카드 타입인 메서드 2개를 선언했습니다.
```java
    public static void main(String[] args) {
        Hospital<Animal> hospital = new Hospital<>();

        List<? extends Animal> animals = hospital.ret1();
        // 정확한 타입을 알 수 없어서 add 할 수 없다!
        animals.add(new Dog(1)); // 컴파일에러
        animals.add(new Animal(1)); // 컴파일에러

        List<? super Animal> objects = hospital.ret2();
        Object object = objects.get(0); // 최상위 타입인 Object 객체로 얻어진다

    }
```
직접 클라이언트 입장에서 반환형이 한정적 와일드카드 타입인 메서드를 사용해보니 이런 문제가 있습니다.
- `<? extends Animal>`을 반환하는 경우
  - Animal을 상속하는 어떤 타입의 리스트지만 정확한 타입을 알 수 없어 add 할 수 없다
- `<? super Animal>`을 반환하는 경우
  - Animal의 부모 클래스인 어떤 타입의 리스트지만 정확한 타입을 알 수 없어 최상위 타입인 Object를 얻는다. 

---

# Comparable과 Comparator 

```java
    public static <E extends Comparable<E>> E max(List<E> c) {
        if (c.isEmpty())
            throw new IllegalArgumentException("컬렉션이 비어 있습니다.");

        E result = null;
        for (E e : c)
            if (result == null || e.compareTo(result) > 0)
                result = Objects.requireNonNull(e);

        return result;
    }
```
위 메서드 선언을 PECS 공식을 적용해 와일드카드 타입을 사용하도록 바꿔보자
```java
    public static <E extends Comparable<? super E>> E max(List<? extends E> c)
```
- c는 E를 생산하는 생산자다. 따라서 `<? extends E>`로 변경했다.
- 타입 매개변수에도 Comparable은 E를 소비해서 선후관계를 뜻하는 정수를 생성한다. 따라서 `<? super E>`로 변경한다. 
```java
public interface Comparable<T> {
    public int compareTo(T o);
}
```
- **Comparable, Comparator는 항상 소비자이므로 `<? super E>`를 사용하자.**

```java
// 변경 전 
public static <E extends Comparable<E>> E max(List<E> c)
// 변경 후
public static <E extends Comparable<? super E>> E max(List<? extends E> c)

List<ScheduledFuture<?>> schedulesFeatures = ...;
```
굳이 이렇게까지 복잡하게 써야 하는 이유를 알아보자
- ScheduledFuture는 `Comparable<SchedulredFuture>`를 구현하지 않는다. ScheduledFuture가 구현하는 인터페이스 Delayed는 `Comparable<Delaye>`를 구현한다. 즉 ScheduledFuture는 자기 자신뿐만 아니라 Delayed와도 비교할 수 있다.
- 변경 전 메서드 선언에서 E는 `Comparable<E>`를 구현해야 한다. ScheduledFuture는 `Comparable<SchedulredFuture>`를 구현하지 않기 때문에 부합하지 않는다.
- 변경 후 메서드 선언에서 E는 자신을 포함한 자신의 상위 클래스 ?의 `Comparable<?>`를 구현하면 된다. ScheduledFuture의 상위 클래스 Delayed가 `Comparable<Delaye>`를 구현하르모 ScheduledFuture가 부합하다.
- 즉 Comparable 또는 Comparator를 직접 구현하지 않고 직접 구현한 상위 타입을 확장한 타입을 지원하기 위해 Comparable, Comparator는 `<? super E>`를 사용하자!

# 타입 매개변수 vs 와일드카드
- 제네릭타입(`List<E>`)를 쓰고 싶지만 실제 타입 매개변수(E)가 무엇인지 신경 쓰고 싶지 않을 때 와일드카드를 사용한다.(`List<?>`)
```java
    public <T> void add1(List<T> list, int i){
        T t = list.get(i);
        list.add(t);
    }

    public void add2(List<?> list, int i){
        Object o = list.get(i);
        list.add(null);
        list.add(o); // 컴파일에러
    }
```
- 타입 매개변수를 사용하면 list의 원소가 T타입이라는 것이 보장된다.
  - T타입 list에서 꺼낸 원소는 T타입이고, T타입 list에 T타입 객체를 넣을 수 있다.
- 와일드카드를 사용하면 list가 무슨 타입의 List인지 모른다.
  - 무슨 타입인지 모르기 때문에 모든 타입의 최상위 타입인 Object로 꺼내진다.
  - 무슨 타입 List인지 모르기 때문에 null 말고는 값을 넣을 수 없다.
- **public API에서는 와일드카드를 사용**하자. 어떤 List이던지 동작할 것이라는 걸 알 수 있고, 단순하다.
  - 메서드 선언에 타입 매개변수(T)가 한번만 나오면 와일드카드(?)로 대체하라
  - 비한정적 타입 매개변수면 비한정적 와일드카드로, 한정적 타입 매개변수면 한정적 와일드카드로 바꾼다.
```java
    public void add2(List<?> list, int i) {
        add(list, i);
    }

    private <T> void add(List<T> list, int i) {
        list.add(list.get(i));
    }
```
- 위에서 설명햇듯이 와일드카드를 사용하면 타입에 상관없이 작동해야 하기 때문에 null을 제외한 어떤 값도 list에 넣을 수 없다
- list에 넣기 위한 private 메서드를 선언하자
- API는 단순한 와일드카드로 유지하면서 컴파일에러를 해결할 수 있다.


