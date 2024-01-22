# 명명 패턴 
- 도구나 프레임워크가 특별히 다뤄야 할 프로그램 요소에 명명 패턴을 적용했었다.
> 명명패턴은 변수, 함수와 같은 프로그램 요소의 이름을 일관된 방식으로 작성하는 것이다

- 예를 들어 JUnit버전 3까지는 테스트 메서드 이름을 test로 시작하게 한다.

## 명명 패턴 단점 
1. 오타가 나면 안된다
   - testMember를 tsetMember로 잘못 작성하면 JUnit3은 이 메서드를 무시한다.
   - 개발자는 테스트가 통과한 것으로 착각할 수 있다.
  
2. 올바른 프로그래밍 요소에서만 사용되리라 보증할 방법이 없다.
   - 메서드 이름을 test로 시작해야 하는데 클래스 이름을 Test로 시작할 수 있다.
   - JUnit3가 TestMember 클래스는 무시한다.
  
3. 프로그램 요소를 매개변수로 전달할 방법이 없다
   - 특정 예외를 던져아 성공하는 테스트
   - 메서드 이름에 예외를 붙여도 컴파일러는 그것이 예외명인지 알 수 없다
  
# 애너테이션 
## 커스텀 애너테이션
```java
@Retention(RetentionPolicy.RUNTIME)
public @interface Okay {
    int love();
}
```
- `자료형 파라미터명()`으로 애너테이션의 파라미터를 선언할 수 있다.
```java
    @Okay(love = 2)
    public void hello() {
    }

    @No(love = 10, hate = 20)
    public void hello2() {
    }
```
- `@애너테이션명(속섬명=값, 속성명=값)`의 형태로 사용한다.
```java
        Class<?> testClass = Class.forName("effectivejava.chapter6.item39.Service");
        for (Method m : testClass.getDeclaredMethods()) {
            Okay annotation = m.getAnnotation(Okay.class);
            int love = annotation.love();
            System.out.println(love);
        }
```
- `getAnnotation(클래스리터럴)` 로 애너테이션을 얻을 수 있다.
- `애너테이션.속성명()`으로 속성값을 얻을 수 있다. 

## 마커애너테이션
애너테이션은 명명 패턴의 단점을 해결해준다. 
```java
@Retention(RetentionPolicy.RUNTIME)
@Target(ElementType.METHOD)
public @interface Test {
}
```
- 메타애너테이션 : 애너테이션 선언에 다는 애너테이션
  - `@Retention(RetentionPolicy.RUNTIME)` : `@Test`가 런타임에도 유지된다. 런타임에 유지되어야 테스트 도구가 인식할 수 있다.
  - `@Target(ElementType.METHOD)` : `@Test`가 메서드에만 사용되어야 한다. 클래스, 필드 등에는 사용할 수 없다.
- 마커애너테이션 : 아무 매개변수 없이 대상에 단순히 마킹하는 애너테이션, `@Test`
- `@Test` 애너테이션을 테스트 대상 메서드에 달아준다. 애너테이션을 사용하면 `@Tset`와 같이 오타가 발생하거나, 필드에 애너테이션을 달면 컴파일 오류가 난다.
- 매개변수 없는 정적 메서드 전용이다. 이것은 컴파일러로 강제할 수는 없지만 지키지 않으면 RunTests로 애너테이션이 달린 메서드를 호출할 때 예외가 발생한다. 

```java
public class Sample {
    @Test
    public static void m1() {
    }        // 성공해야 한다.

    public static void m2() {
    }

    @Test
    public static void m3() {    // 실패해야 한다.
        throw new RuntimeException("실패");
    }

    public static void m4() {
    }  // 테스트가 아니다.

    @Test
    public void m5() {
    }   // 잘못 사용한 예: 정적 메서드가 아니다.

    public static void m6() {
    }

    @Test
    public static void m7() {    // 실패해야 한다.
        throw new RuntimeException("실패");
    }

    public static void m8() {
    }
}
```
- 테스트하고자 하는 메서드에 `@Test` 애너테이션을 달 수 있다.
- 애너테이션을 단 것 자체로는 동작에 아무 영향이 없다. 애너테이션에 관심 있는 도구에서 애너테이션이 달린 메서드 대상으로 특별한 처리를 할 수 있다.

```java
public class RunTests {
    public static void main(String[] args) throws Exception {
        int tests = 0;
        int passed = 0;
        Class<?> testClass = Class.forName("effectivejava.chapter6.item39.Sample");
        for (Method m : testClass.getDeclaredMethods()) {
            if (m.isAnnotationPresent(Test.class)) {
                tests++;
                try {
                    m.invoke(null);
                    passed++;
                } catch (InvocationTargetException wrappedExc) {
                    Throwable exc = wrappedExc.getCause();
                    System.out.println(m + " 실패: " + exc);
                } catch (Exception exc) {
                    System.out.println("잘못 사용한 @Test: " + m);
                }
            }
        }
        System.out.printf("성공: %d, 실패: %d%n",
                passed, tests - passed);
    }
}
```
- 애너테이션에 관심 있는 도구가 RunTests이다. `isAnootationPresent`로 Test 애너테이션이 달린 메서드만 걸러낸다
- 테스트 대상인 메서드를 호출하고 대상 메서드에서 예외를 던질 경우 `InovcationTargetException`으로 감싸서 던진다.
- 그 외 예외는 `@Test`를 잘못 사용한 것이다. (인스터느 메서드, 매개변수가 있는 메서드 등.. )

## 매개변수 1개를 받는 애너테이션 
```java
@Retention(RetentionPolicy.RUNTIME)
@Target(ElementType.METHOD)
public @interface ExceptionTest {
    Class<? extends Throwable> value();
}
```
- 모든 예외타입의 클래스 리터럴을 파라미터로 갖는 애너테이션이다.
```java
    @ExceptionTest(ArithmeticException.class)
    public static void m1() {  // 성공해야 한다.
        int i = 0;
        i = i / i;
    }
```
- 애너테이션의 파라미터로 원하는 예외타입의 클래스 리터럴을 전달한다.

```java
            if (m.isAnnotationPresent(ExceptionTest.class)) {
                tests++;
                try {
                    m.invoke(null);
                    System.out.printf("테스트 %s 실패: 예외를 던지지 않음%n", m);
                } catch (InvocationTargetException wrappedEx) {
                    Throwable exc = wrappedEx.getCause();
                    Class<? extends Throwable> excType =
                            m.getAnnotation(ExceptionTest.class).value();
                    if (excType.isInstance(exc)) {
                        passed++;
                    } else {
                        System.out.printf(
                                "테스트 %s 실패: 기대한 예외 %s, 발생한 예외 %s%n",
                                m, excType.getName(), exc);
                    }
                } catch (Exception exc) {
                    System.out.println("잘못 사용한 @ExceptionTest: " + m);
                }
            }
        }
```
- `m.getAnnotation(ExceptionTest.class).value();`로 애너테이션의 매개변수 값을 가져온다.
- 매개변수로 전달한 타입의 예외를 던져야야 성공한 테스트로 취급한다

## 배열 매개변수를 받는 애너테이션 
```java
@Retention(RetentionPolicy.RUNTIME)
@Target(ElementType.METHOD)
public @interface ExceptionTest {
    Class<? extends Exception>[] value();
}
```
- 예외 클래스 리터럴의 배열을 파라미터로 갖는 애너테이션이다.

```java
    @ExceptionTest({ IndexOutOfBoundsException.class,
                     NullPointerException.class })
    public static void doublyBad() {   // 성공해야 한다.
        List<String> list = new ArrayList<>();

        // 자바 API 명세에 따르면 다음 메서드는 IndexOutOfBoundsException이나
        // NullPointerException을 던질 수 있다.
        list.addAll(5, null);
    }
```
- 매개변수를 하나만 받을수도 있고, 위처럼 배열 형태로 받을수도 있다.

```java
if (m.isAnnotationPresent(ExceptionTest.class)) {
                tests++;
                try {
                    m.invoke(null);
                    System.out.printf("테스트 %s 실패: 예외를 던지지 않음%n", m);
                } catch (Throwable wrappedExc) {
                    Throwable exc = wrappedExc.getCause();
                    int oldPassed = passed;
                    Class<? extends Throwable>[] excTypes =
                            m.getAnnotation(ExceptionTest.class).value();
                    for (Class<? extends Throwable> excType : excTypes) {
                        if (excType.isInstance(exc)) {
                            passed++;
                            break;
                        }
                    }
                    if (passed == oldPassed)
                        System.out.printf("테스트 %s 실패: %s %n", m, exc);
                }
            }
```
- `Class<? extends Throwable>[] excTypes =
                            m.getAnnotation(ExceptionTest.class).value();` 애너테이션의 파라미터 값을 배열로 가져온다.
- 명시한예외 중에서 예외가 발생하면 테스트 통과

## @Repeatable 애너테이션 
- 배열 매개변수 대신 애너테이션에 메타애너테이션 `@Repeatable`을 달 수도 있다. 
- `@Repeatable`을 단 애너테이션은 하나의 프로그램 요소에 여러번 달 수 있다.

```java
@Retention(RetentionPolicy.RUNTIME)
@Target(ElementType.METHOD)
@Repeatable(ExceptionTestContainer.class)
public @interface ExceptionTest {
    Class<? extends Throwable> value();
}

@Retention(RetentionPolicy.RUNTIME)
@Target(ElementType.METHOD)
public @interface ExceptionTestContainer {
    ExceptionTest[] value();
}
```
- 컨테이너 애너테이션을 만들어야 한다. 그리고 `@Repeatable(컨테이너애너테이션.class)`와 같이 선언해준다
- 컨테이너 애너테이션은 정의하려는 애너테이션의 배열을 반환하는 value 메서드를 선언한다.
- 컨테이너 애너테이션에 적절한 Retention과 Target을 명시한다.

```java
    @ExceptionTest(IndexOutOfBoundsException.class)
    @ExceptionTest(NullPointerException.class)
    public static void doublyBad() {
        List<String> list = new ArrayList<>();

        // 자바 API 명세에 따르면 다음 메서드는 IndexOutOfBoundsException이나
        // NullPointerException을 던질 수 있다.
        list.addAll(5, null);
    }
```
- `@Repeatable`을 단 애너테이션은 이렇게 반복해서 사용할 수 있다 .
```java
if (m.isAnnotationPresent(ExceptionTest.class)
                    || m.isAnnotationPresent(ExceptionTestContainer.class)) {
                tests++;
                try {
                    m.invoke(null);
                    System.out.printf("테스트 %s 실패: 예외를 던지지 않음%n", m);
                } catch (Throwable wrappedExc) {
                    Throwable exc = wrappedExc.getCause();
                    int oldPassed = passed;
                    ExceptionTest[] excTests =
                            m.getAnnotationsByType(ExceptionTest.class);
                    for (ExceptionTest excTest : excTests) {
                        if (excTest.value().isInstance(exc)) {
                            passed++;
                            break;
                        }
                    }
                    if (passed == oldPassed)
                        System.out.printf("테스트 %s 실패: %s %n", m, exc);
                }
            }
```
반복 가능 애너테이션을 처리할 때는 주의해야 한다. 
- 반복 가능 애너테이션은 반복해서 달면 컨테이너 애너테이션이 적용된다
- getAnnotationsByType을 컨테이너 애너테이션과 그냥 애너테이션 구분 없이 가져온다.
- isAnnotationPresent는 이것을 구분해서 가져온다.

---

소스코드에 추가 정보를 제공하는 도구를 위해 명명 패턴이 아닌 애너테이션을 사용하는 것이 좋다. 




