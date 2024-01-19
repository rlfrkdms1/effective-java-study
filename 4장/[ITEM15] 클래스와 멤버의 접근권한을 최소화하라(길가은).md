잘 설계된 컴포넌트는 모든 내부 구현을 완벽히 숨겨, **구현과 API**를 깔끔히 **분리**한다. 

캡슐화는 다음의 장점들을 가진다. 
- 시스템 개발 속도를 높인다. 
  - 여러 컴포넌트를 병렬로 개발할 수 있기 때문이다. 
- 시스템 관리 비용을 낮춘다. 
  - 각 컴포넌트를 더 빨리 파악하여 디버깅할 수 있고, 다른 컴포넌트로 교체하는 부담도 적기 때문이다.  
- 성능 최적화에 도움을 준다. 
  - 최적화할 컴포넌트만 최적화할 수 있기때문이다. 
- 소프트웨어 재사용성을 높인다. 
  - 의존성이 낮고 독자적으로 동작 -> 낯선 환경에서도 유용하게 쓰일 가능성이 크다. 
- 큰 시스템을 제작하는 난이도를 낮춰준다. 
  - 전체가 완성되지 않아도 개별 컴포넌트의 동작을 검증 가능
  
## 접근 제한자
`private, protected, public `
> 모든 클래스와 멤버의 접근성을 가능한 한 좁혀야한다. 즉, SW가 올바로ㅗ 동작하는 한 항상 가장 낮은 접근 수준을 부여해야한다. 

### 톱레벨 클래스와 인터페이스에 부여할 수 있는 수준
|접근 제한자|범위|
|--------|---|
|public|모든 곳에서 접근할 수 있다. |
|package-private|범위 포함, 이 멤버를 선언한 클래스의 하위 클래스에서도 접근할 수 있다.|

public 으로 선언 -> 공개 API가 되므로 패키지 외부에서 사용할 일이 없다면 package-private으로 선언해서 사용해야한다. 
package-private 클래스 안에 private static으로 중첩 시킨다면, 바깥 클래스하나에서만 접근할 수 있다. 

```java
class Outside{
    public void loud(){
        new Inside().voice();
    }
}
class Inside{
    public void voice(){
        System.out.println("inside");
    }
}
```
위와 같이 Inside 클래스는 Outside 클래스 내에서만 사용된다면, Outside 내에 private static으로 중첩 시켜 사용해 Outside에서만 사용가능 하도록 만드는 것이다. 

```java
class Outside{
    public void loud(){
        new Inside().voice();
    }
    
    private static class Inside{
        public void voice(){
            System.out.println("inside");
        }
    }
}
```

- pulbic 클래스 -> 해당 패키지의 API
- package-private 클래스 -> 내부 구현

=>필요없는 public 클래스를 package-private로 좁혀야함

### 멤버에 부여할 수 있는 수준
|접근 제한자|범위|
|--------|---|
|public|모든 곳에서 접근할 수 있다. |
|protected|package-private 범위 포함, 이 멤버를 선언한 클래스의 하위 클래스에서도 접근할 수 있다.|
|package-private|멤버가 소속된 패키지 않의 모든 클래스에서 접근 가능|
|private|멤버를 선언한 톱레벨 클래스에서만 접근 가능|

### 컴포넌트 설계
1. 공개 API를 세심히 설계
2. 그 외 모든 멤버 private
3. 오직 같은 패키지의 다른 클래스가 접근해야 하는 멤버에 한해 package-private으로 풀어준다. 

> 권한을 풀어주는 일을 자주 하게 된다면 컴포넌트를 더 분해해야하는 것으 ㄴ아닌지 다시 고민해보자. 

public 클래스의 protected 멤버는 공개 API이므로 영원히 지원돼야 하므로 적을 수록 좋다. 

## 제약
> 상위 클래스의 메서드를 재정의할 때는 그 접근 수준을 상위 클래스에서보다 좁게 설정할 수 없다. 

이는 리스코프 치환 원칙을 지키기 위해 필요하다. 리스코프 치환 원칙이란,  자료형 S가 자료형 T의 서브타입라면 필요한 프로그램의 속성(정확성, 수행하는 업무 등)의 변경 없이 자료형 T의 객체를 자료형 S의 객체로 교체(치환)할 수 있어야 한다는 원칙이다.

```java
 public class Upper {
    
    public void targetToOverride() {
        System.out.println("접근 수준을 더 좁게 오버라이딩 할 수 없다. ");
    }
}
```
위와 같이 상위 클래스가 있고, 하위 클래스에서 이를 상속받아 `targetToOverride()`를 private으로 오버라이드 하고 싶다면 컴파일에러가 뜬다. 

```java

public class Lower extends Upper{
    @Override
    private void targetToOverride() {
        super.targetToOverride();
    }
}
```
```
'targetToOverride()' in 'CH4.Lower' clashes with 'targetToOverride()' in 'CH4.Upper'; attempting to assign weaker access privileges ('private'); was 'public'
```

![](https://velog.velcdn.com/images/rlfrkdms1/post/3e351786-c6e4-4436-8324-34892a6d6e82/image.png)

```java
public interface Example {
    void test();
}

public class ExampleImpl implements Example {
    @Override
    private void test() {

    }
}
```
클래스가 인터페이스를 구현하는 건 이 규칙의 특별한 예로 볼 수 있고, 이때 클래스는 인터페이스가 정의한 모든 메서드를 public으로 선언해야 한다. 
![](https://velog.velcdn.com/images/rlfrkdms1/post/42cda88f-0387-42a1-9d7f-1533543e5094/image.png)

단지 코드를 테스트할 목적으로 접근 범위를 넓히려할 때는 적당한 수준까지만 넓혀야 한다. 즉, 테스트만을 위해 클래스, 인터페이스, 멤버를 공개 API로 만들어서는 안된다. 

## public 필드
>public 클래스의 인스턴스 필드는 되도록 public이 아니어야 한다. 

필드가 가변 객체를 참조하거나, final이 아닌 인스턴스 필드를 public으로 선언하면 그 필드에 담을 수 있는 값을 제한할 힘을 잃게 된다. 그 필드와 관련된 모든 것은 불변식을 보장할 수 없게 된다는 뜻이다. 

=> public 가변 필드를 갖는 클래스는 일반적으로 스레드 안전하지 않다. 

그렇다면 public final 로 바꾸면 되지 않을까? 생각할 수 있지만, 내부 구현을 바꾸고 싶어도 해당 필드를 없애는 방식으로는 리팩터링할 수 없게 된다. 이러한 문제는 정적 필드에서도 마찬가지이나, 예외가 하나 있다. 

해당 클래스가 표현하는 추상 개념을 완성하는 데 꼭 필요한 구성요소로써의 상수라면 `public static final`필드로 공개해도 좋으며 상수형태로 사용한다. 

```java
public static final int MAX = 60;
```
이러한 필드는 반드시 기본 타입 값이나, 불변 객체를 참조해야한다. 

## 배열
>클래스에서 public static final 배열 필드를 두거나 이 필드를 반환하는 접근자 메서드를 제공해서는 안된다. 

```java
public static final Thing[] VALUES = {...};
```
위의 코드는 문제를 일으킨다. 가변 객체인 배열을 참조했기 때문이다. 
```java
    @Test
    void public_final_배열() {
        Example.VALUES[2] = 0;
        Assertions.assertThat(Example.VALUES[2]).isEqualTo(0);
        Assertions.assertThat(Example.VALUES[2]).isNotEqualTo(3);
    }

    class Example{
        public static final int[] VALUES = {1, 2, 3};
    }
```
### 해결책

#### 불변 리스트 추가
```java
private static final Thing[] PRIVATE_VALUES = {...};
public static final List<Thing> VALUES = Colloctions.unmodifiableList(Arrays.asList(PRIVATE_VALUES));
```
public이었던 배열을 private으로 숨기고 해당 필드에 접근할 때는 해당 배열을 불변 리스트로 복제해서 건네 주는 것이다. 

#### 복사본 반환
```java
private static final Thing[] PRIVATE_VALUES = {...};
public static final Thing[] values(){
    return PRIVATE_VALUES.clone();
}
```
`clone` 은 방어적 복사를 해 clone한 배열을 수정해도 원본 배열이 수정되지 않는다. Java의 복사에 대해선 [이 글](https://velog.io/@rlfrkdms1/자바의-배열을-사용하면서-조심해야-할-복사#clone)에 정리해두었다. 

## 모듈
모듈은 자신이 속하는 패키지 중 공개할 것들을 선언한다. protected 혹은 public 멤버라도 해당 패키지를 공개하지 않았다면 모듈 외부에서는 접근할 수 없다. 

모듈이 공개했는지 여부와 상관없이, public 클래스가 선언한 모든 public 혹은 protected 멤버를 모듈 밖에서도 접근할 수 있게 되므로 주의해서 사용해야 한다. 

## 핵심 정리

>프로그램 요소의 접근성은 가능한 한 최소한으로 하라. 

꼭 필요한 것만 골라 최소한의 public API를 설계하자. 그 외에는 클래스, 인터페이스, 멤버가 의도치 않게 API로 공개되는 일이 없도록 해야한다. public 클래스는 상수용 public static final 필드 외에는 어떠한 public 필드도 가져가서는 안된다. public sattic final 필드가 참조하는 객체가 불변인히 확인하라

#### 출처

이펙티브 자바 3/E

[리스코프 치환 원칙](https://ko.wikipedia.org/wiki/리스코프_치환_원칙)


