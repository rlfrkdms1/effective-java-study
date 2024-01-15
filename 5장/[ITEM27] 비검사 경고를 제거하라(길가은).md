제네릭 사용시 마주치게 되는 경고
- 비검사 형변환 경고
- 비검사 메서드 호출 경고
- 비검사 매개변수화 가변인수 타입 경고
- 비검사 변환 경고


아래와 같이 입력해보자. 
```java
Set<Stamp> test = new HashSet();
```

그럼 경고들이 뜨는데, 경고들을 살펴보자. 
```
Raw use of parameterized class 'HashSet'
Unchecked assignment: 'java.util.HashSet' to 'java.util.Set<item26.Stamp>'
```
또한 `<>`연산자를 삽입하면 된다고 뜬다. 따라서 아래와 같이 수정해보면 경고들은 모두 사라진다. 

```java
Set<Stamp> test = new HashSet<>();
```
`<>` 연산자를 붙이면 컴파일러가 올바른 실제 타입 매개변수를 추론해주기 때문이다. 

>할 수 있는 한 모든 비검사 경고를 제거해야한다. 

모두 제거한다면, 타입 안정성이 보장된다. 즉, 런타임에 ClassCastException이 발생할 일이 없다. 

경고를 제거할 수는 없지만 타입 안전하다고 확신할 수 있다면 `@SuppressWarnings("unchecked")` 애너테이션을 달아 경고를 숨기자. 
단, 타입 안정함을 검증하지 않은 채 경고를 숨기면 스스로에게 잘못된 보안 인식을 심어주는 꼴이다. 이 애너테이션은 개별 지역변수 선언부터 클래스 전체까지 어떤 선언에도 달 수 있다. 

하지만, `@SuppressWarnings` 애너테이션은 항상 가능한 한 **좁은 범위**에 적용하자. 심각한 경고를 놓칠 수 있으니, 절대 클래스 전체에 적용해서는 안된다. 

`ArrayList`의 `toArray()`를 살펴보자. 

```java
    @SuppressWarnings("unchecked")
    public <T> T[] toArray(T[] a) {
        if (a.length < size)
            // Make a new array of a's runtime type, but my contents:
            return (T[]) Arrays.copyOf(elementData, size, a.getClass());
        System.arraycopy(elementData, 0, a, 0, size);
        if (a.length > size)
            a[size] = null;
        return a;
    }
```
`ArrayList`를 컴파일하면 이 메서드에선 `return (T[]) Arrays.copyOf(elementData, size, a.getClass());`에서 경고가 발생한다. 애너테이션은 선언문에만 달 수 있기에 return문에 달지 못한다. 따라서 다음과 같이 수정해야한다. 

```java
    public <T> T[] toArray(T[] a) {
        if (a.length < size)
            @SuppressWarnings("unchecked")
            (T[]) result = (T[]) Arrays.copyOf(elementData, size, a.getClass());
            return result;
        System.arraycopy(elementData, 0, a, 0, size);
        if (a.length > size)
            a[size] = null;
        return a;
    }
```
지역변수 result를 선언해 해당 변수에 애너테이션을 붙인 것을 볼 수 있다. 이처럼 경고를 무시하는 이 애너테이션은 항상 최소한의 범위에만 달아야 한다. 

또한, 이 애너테이션을 사용할 때면 그 경고를 무시해도 안전한 이유를 항상 주석으로 남겨야 한다. 

다른 사람이 그 코드를 이해하는 데 도움이 되며, 더 중요하게는, 다른 사람이 그 코드를 잘못 수정해 타입 안전성을 잃는 상황을 줄여준다. 

타입 안전성이란? 
타입에 불안정적이다 라고 하는것은 타입을 판별(Type Check) 하지 못해 Runtime 시 타입으로 인한 문제가 발생하는 것입니다.
Type Safe 하다 라는 것은 그 반대로 타입을 판별(Type Check) 할 수 있어 Runtime시가 아닌 컴파일시 문제를 잡을 수 있는 것입니다.

#### 출처

[타입 안전성](https://dololak.tistory.com/17)
이펙티브 자바 3/E
