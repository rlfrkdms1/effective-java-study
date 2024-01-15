# 타입 안전 이종 컨테이너
- `Set<E>`에는 E만 저장할 수 있다
- 유연하게 Set에 다양한 타입을 저장하고 싶을 수 있다.
- 타입 안전 이종 컨테이너는 타입 안전성이 보장되며 서로 다른 타입을 담을 수 있는 컨테이너이다.

```java
public class FlexibleMap {

    private Map<Class<?>, Object> map = new HashMap<>();

    public <T> void put(Class<T> type, T instance) {
        map.put(Objects.requireNonNull(type), instance);
    }

    public <T> T get(Class<T> type) {
        return type.cast(map.get(type));
    }
}
```
- 컨테이너 대신 key를 매개변수화한다.
- 서로 다른 매개변수화 타입 (`Class<String>, Class<Integer>` 등.. ) 이 키가 될 수 있으므로 다양한 타입을 지원한다.
- map의 value는 Object 타입으로 map의 선언상 value가 반드시 key에 맞는 타입을 보장하지 않는다. (key가 `Class<String>`이라고 해서 value가 String 타입임을 보장하지 않는다)
- get에서 주어진 Class 타입에 해당하는 값을 꺼낸다.
  ```java
      public T cast(Object obj) {
        if (obj != null && !isInstance(obj))
            throw new ClassCastException(cannotCastMsg(obj));
        return (T) obj;
    }
  ```
  - `Class<T>`의 메서드 cast는 위와 같다. 즉 `Class<String>.cast`는 대상을 String과 같은 타입인지 검사하고 String으로 캐스팅한다.
```java
    public static void main(String[] args) {
        FlexibleMap map = new FlexibleMap();
        map.put(Integer.class, 10);
        map.put(String.class, "love");
        System.out.println(map.get(Integer.class));
        System.out.println(map.get(String.class));
    }
```
- `Integer.class`나 `String.class`는 각 타입의 class 리터럴이다. class 리터럴은 제네릭 클래스다. 즉 `Class<String>`, `Class<Integer>` 타입이다. class 리터럴을 type token이라 한다.
- 타입 안전 이종 컨테이너의 역할을 잘 수행한다

## 제약
FlexibleMap에는 제약이 두가지 있다.
1. 악의적인 클라이언트가 Class 객체를 제네릭이 아닌 로 타입으로 넘기면 타입 안정성이 깨진다.
```java
        map.put((Class)Integer.class, new Robot());
        Integer no = map.get(Integer.class); // ClassCastException
```
- Integer.class를 Class(로 타입)으로 캐스팅해서 전달한다. 로 타입이므로 매개변수 `Class<T>`에서 `<T>`를 무시하므로 컴파일에러가 발생하지 않는다 
- 런타임에 ClassCastException을 던진다
```java
    public <T> void put(Class<T> type, T instance) {
        map.put(Objects.requireNonNull(type), type.cast(instance));
    }
```
- put을 위와 같이 수정해서 map이 불변식을 보장하게 할 수 있다.(여전히 컴파일은 되고 런타임에 ClassCastException이 발생한다)

2. 실체화 불가 타입은 타입 안전 이종 컨테이너에 사용할 수 없다
   - 실체화 불가 타입인 대부분 제네릭과 매개변수화 타입(`List<String>`)은 사용할 수 없다.
   - 실체화 불가 타입은 Class 객체를 얻을 수 없다.
   - 슈퍼 타입 토큰을 사용하면 해결할 수 있지만 이 방식도 한계가 있다.

---

## 한정적 타입 토큰 

```java
    public <T extends Robot> void put(Class<T> type, T instance) {
        map.put(Objects.requireNonNull(type), type.cast(instance));
    }
```
- 한정적 타입 토큰을 사용해 타입 토큰의 종류를 제한할 수 있다.

### 애너테이션 
애너테이션 API가 한정적 타입 토큰을 적극사용한다.
```java
<T extends Annotation> T getAnnotation(Class<T> annotationClass);
```
- 타입 토큰 annotationClass는 Annotation을 상속한 클래스의 타입 토큰으로 제한된다. 

```java
public class PrintAnnotation {
    static Annotation getAnnotation(AnnotatedElement element,
                                    String annotationTypeName) {
        Class<?> annotationType = null; // 비한정적 타입 토큰
        try {
            annotationType = Class.forName(annotationTypeName);
        } catch (Exception ex) {
            throw new IllegalArgumentException(ex);
        }
        return element.getAnnotation(
                annotationType.asSubclass(Annotation.class));
    }
```
한정적 타입 토큰을 받는 메서드에 비한정적 타입 토큰을 전달하려면 어떻게 할까? 예를 들어 `Class<?>`를 `Class<T extends Annotation>`에 전달하고 싶다.
- `Class<? extends Annotation>`으로 형변환 할 수 있지만 비검사이므로 경고가 뜬다
- Class.asSubClass 메서드를 사용하자. 자신의 Class 객체를 명시한 클래스로(인자로 받은 타입으로) 형변환해준다.
  - 자신이 명시한 클래스의 하위 타입이면 형변환 성공, 아니라면 ClassCastException을 던다






