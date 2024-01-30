API를 쓸모 있게 하려면 잘 작성된 문서도 곁들여야한다. 

자바는 자바독을 지원해 소스코드 파일에서 문서화 주석이라는 형태로 기술된 설명을 추려 API 문서로 변환해준다. 

> API를 올바로 문서화하려면 공개된 모든 클래스, 인터페이스, 메서드, 필드 선언ㄴ에 문서화 주석을 달아야한다. 

직렬화할 수 있는 클래스 -> 직렬화 형태에 관해서도 작성

문서화 주석이 없다면 자바독도 요소들의 선언만 나열해준다. 문서가 잘 갖춰지지 않은 API는 오류의 원인이 되기 쉽다. 

기본생성자에는 문서화 주석을 달 수 없으니, 공개 클래스는 절대 기본 생성자를 사용하면 안된다. 

유지보수 까지 고려한다면 대다수의 공개되지 않은 클래스, 인터페이스, 생성자, 메서드, 필드에도 문서화 주석을 달아야한다. 

> 메서드용 문서화 주석에는 해당 메서드와 클라이언트 사이의 규약을 명료하게 기술해야한다. 

상속용으로 설계된 클래스의 메서드가 아니라면 how가 아닌 what을 기술해야한다. 

문서화 주석에는 메서드를 호출하기 위한 전제조건, 메서드가 성공적으로 수행된 후에 만족해야 하는 사후 조건도 모두 나열해야한다. 

- `@throws` : 전제조건. 비검사 예외를 선언해 암시적으로 기술
- `@param` : 조건에 영향받는 매개변수에 기술

부작용도 문서화해야하는데, 부작용은 사후조건으로 명확히 나타나지는 않지만 시스템의 상태에 어떠한 변화를 가져오는 것을 뜻한다. 

- 메서드의 계약을 완벽히 기술 -> 모든 매개변수에 `@param`
- 반환 타입이 void가 아니라면 -> `@return`
- 발생할 가능성이 있는 모든 예외 -> `@throws`

`@return`, `@param` : 관례상 명사구
`@throws` : if로 시작해 해당 예외를 던지는 조건을 설명하는 절이 뒤따른다. 

모두 마침표를 붙이지 않는다. 

```java
    /**
     * Returns the element at the specified position in this list.
     *
     * <p>This method is <i>not</i> guaranteed to run in constant
     * time. In some implementations it may run in time proportional
     * to the element position.
     *
     * @param  index index of element to return; must be
     *         non-negative and less than the size of this list
     * @return the element at the specified position in this list
     * @throws IndexOutOfBoundsException if the index is out of range
     *         ({@code index < 0 || index >= this.size()})
     */
    E get(int index) {
        return null;
    }
```

- 우리말
```java
     /**
      * 이 리스트에서 지정한 위치의 원소를 반환한다.
      *
      * <p>이 메서드는 상수 시간에 수행됨을 보장하지 <i>않는다</i>. 구현에 따라
      * 원소의 위치에 비례해 시간이 걸릴 수도 있다.
      *
      * @param  index 반환할 원소의 인덱스; 0 이상이고 리스트 크기보다 작아야 한다.
      * @return 이 리스트에서 지정한 위치의 원소
      * @throws IndexOutOfBoundsException index가 범위를 벗어나면,
      * 즉, ({@code index < 0 || index >= this.size()})이면 발생한다.
      */
     E get(int index) {
         return null;
     }
```
자바독은 문서화 주석을 HTML로 변환하기 때문에 주석안의 HTML요소들이 최종 HTML문서에 반영된다. 

- `@code` 
  - 태그로 감싼 내용을 코드용 폰트로 렌더링
  - 태그로 감싼 애용에 포함된 HTML요소나 다른 자바독 태그 무시
  - 여러줄로 넣으려면 `<pre>{@code ... }</pre>`형태로 사용
- `this list`
  - 인스턴스 메서드의 문서화 주석에 쓰인 `this`: 호출된 메서드가 자리하는 객체
  
재정의하는 경우 재정의 하는 방법을 알려줘야한다. 
- `@implSpec`
  - 자기사용 패턴시 사용
  - 해당 메서드와 하위 클래스 사이의 계약
  - 하위 클래스들이 그 메서드를 상속하거나 super 키워드를 이용해 호출할 때 그 메서드가 어떻게 동작하는지를 명확히 인지하고 사용하도록 해줘야함

```java
    /**
     * Returns true if this collection is empty.
     *
     * @implSpec This implementation returns {@code this.size() == 0}.
     *
     * @return true if this collection is empty
     */
    public boolean isEmpty() {
        return false;
    }
```

- 한글

```java
     /**
      * 이 컬렉션이 비었다면 true를 반환한다.
      *
      * @implSpec 이 구현은 {@code this.size() == 0}의 결과를 반환한다.
      *
      * @return 이 컬렉션이 비었다면 true, 그렇지 않으면 false
      */
     public boolean isEmpty() {
         return false;
     }
```

자바독 명령줄에서 `-tag "implSpec:a:Implementation Requirements:"` 스위치를 켜주지 않으면 `@implSpec` 태그를 무시한다. 

API 설ㄹ명에 HTML메타 문자(`<,>,&`등)을 포함시키려면 특별한 처리를 해줘야한다. 가장 좋은 방법은 `{@literal}` 로 감싸는 것이다. 

```java
* A geometric series converges if {@literal |r| < 1}.
* 한글
* {@literal |r| < 1}이면 기하 수열이 수렴한다.
```
위 주석은 `|r| < 1이면 기하 수열이 수렴한다.`로 변환된다. 

`* {@literal |r| < 1}이면 기하 수열이 수렴한다.`을 `* |r| {@literal <} 1}이면 기하 수열이 수렴한다.`로 `<`만 감싸게 할 수 있지만 그렇게 하면 코드상 읽기 어려워진다. 

API문서, 코드 양쪽 다 읽기 쉬워야하지만, 양쪽 다 만족하기 어렵다면 API 문서상의 가독성을 우선시 하자.

각 문서화 주석의 첫번째 문장은 요약설명으로 간주된다. 

```java
     /**
      * 이 리스트에서 지정한 위치의 원소를 반환한다.
      *
      * <p>이 메서드는 상수 시간에 수행됨을 보장하지 <i>않는다</i>. 구현에 따라
      * 원소의 위치에 비례해 시간이 걸릴 수도 있다.
      *
      * @param  index 반환할 원소의 인덱스; 0 이상이고 리스트 크기보다 작아야 한다.
      * @return 이 리스트에서 지정한 위치의 원소
      * @throws IndexOutOfBoundsException index가 범위를 벗어나면,
      * 즉, ({@code index < 0 || index >= this.size()})이면 발생한다.
      */
     E get(int index) {
         return null;
     }
```
위의 코드에서 `이 리스트에서 지정한 위치의 원소를 반환한다.`이 이에 해당한다. 

요약설명은 반드시 대상의 기능을 고유하게 기술해야한다.
>한 클래스(혹은 인터페이스)안에서 요약 설명이 똑같은 멤버(혹은 생성자)가 둘 이상이면 안된다. 

다중 정의된 메서드들의 설명은 같은 문장으로 시작하는게 자연스럽겠지만 문서화 주석에서는 허용되지 않는다. 

요약설명이 끝나는 판단은 처음 발견되는 `{<마침표><공백><다음 문장 시작>}` 패턴의 `<마침표>`까지이다. 
- `<공백>` : 스페이스, 탭, 줄바꿈
- `<다음 문장 시작>` : 소문자가 아닌 문자

`머스터드 대령이나 Mrs. 피콕 같은 용의자.`라면 `머스터드 대령이나 Mrs.`을 요약문으로 인식한다. 마침표 다음에 온 **피**가 소문자가 아니고 **Mrs**뒤에 마침표가 왔기 때문이다. 따라서 `{@literal}` 을 사용해야한다. 

```java
    /**
     * A suspect, such as Colonel Mustard or {@literal Mrs. Peacock}.
     */
    public enum Suspect {
        MISS_SCARLETT, PROFESSOR_PLUM, MRS_PEACOCK, MR_GREEN, COLONEL_MUSTARD, MRS_WHITE
    }
```
```java
     /**
      * 머스타드 대령이나 {@literal Mrs. 피콕} 같은 용의자.
      */
     public enum Suspect {
         MISS_SCARLETT, PROFESSOR_PLUM, MRS_PEACOCK, MR_GREEN, COLONEL_MUSTARD, MRS_WHITE
     }
```
자바 10부터는 `{@summary}`가 생겨 다음과 같이 깔끔하게 처리할 수 있다. 
```java
     /**
      * {@summary머스타드 대령이나 Mrs. 피콕 같은 용의자.}
      */
     public enum Suspect {
         MISS_SCARLETT, PROFESSOR_PLUM, MRS_PEACOCK, MR_GREEN, COLONEL_MUSTARD, MRS_WHITE
     }
```

메서드와 생성자의 요약 설명은 해당 메서드와 생성자의 동작을 설명하는 동사구여야한다. 

```
• ArrayList(int initialCapacity): Constructs an empty list with the specified initial capacity.
• Collection.size(): Returns the number of elements in this collection.
```

우리말
```
• Arrayist(int initialCapacity): 지정한 초기 용량을 갖는 빈 리스트를 생 성한다.
• Collection.size(): 이 컬렉션 안의 원소 개수를 반환한다.
```

위 예처럼 2인칭 문장이 아닌 3인칭 문장을 사용해야 한다. 한글에서는 차이가 없다. 

클래스, 인터페이스, 필드의 요약 설명은 대상을 설명하는 명사절이어야한다. 
- 클래스, 인터페이스의 대상 : 인스턴스
- 필드의 대상 : 필드 자신

```
• Instant: An instantaneous point on the time-line.
• Math.PI: The double value that is closer than any other to pi, the ratio of the circumference of a circle to its diameter.
```

우리말
```
• Instant: 타임라인상의 특정 순간(지점)
• Math.PI: 원주율(pi)에 가장 가까운 double 값
```

자바9부터는 문서에 검색 기능이 추가되엇으며 클래스, 메서드, 필드 같은 API 요소의 색인은 자동으로 만들어진다. 원한다면 `{@index}` 태그를 사용해 API에서 중요한 용어를 추가로 색인화할 수 있다. 

```java
* This method complies with the {@index IEEE 754} standard.
```
우리말
```java
* 이 메서드는 {@index.IEEE 754} 표준을 준수한다.
```

> 제네릭 타입이나 제네릭 메서드를 문서화할 때는 모든 타입 매개변수에 주석을 달아야한다. 

```java
/**
 * An object that maps keys to values.  A map cannot contain duplicate keys;
 * each key can map to at most one value.
 ...
 * @param <K> the type of keys maintained by this map
 * @param <V> the type of mapped values
 *
 * @author  Josh Bloch
 * @see HashMap
 * @see TreeMap
 * @see Hashtable
 * @see SortedMap
 * @see Collection
 * @see Set
 * @since 1.2
 */
public interface Map<K, V> { ... }
```

위와 같이 모든 타입 매개변수에 주석을 달아야한다. 
```java
 * @param <K> the type of keys maintained by this map
 * @param <V> the type of mapped values
```

> 열거 타입을 문서화할 때는 상수들에도 주석을 달아야한다. 

```java
    /**
     * An instrument section of a symphony orchestra.
     */
    public enum OrchestraSection {
        /** Woodwinds, such as flute, clarinet, and oboe. */
        WOODWIND,

        /** Brass instruments, such as french horn and trumpet. */
        BRASS,

        /** Percussion instruments, such as timpani and cymbals. */
        PERCUSSION,

        /** Stringed instruments, such as violin and cello. */
        STRING;
    }
```

### 애너테이션 타입을 문서화할 때는 멤버들에도 모두 주석을 달아야한다. 

```java
    /**
     * Indicates that the annotated method is a test method that
     * must throw the designated exception to pass.
     */
    @Retention(RetentionPolicy.RUNTIME)
    @Target(ElementType.METHOD)
    public @interface ExceptionTest {
        /**
         * The exception that the annotated test method must throw
         * in order to pass. (The test is permitted to throw any
         * subtype of the type described by this class object.)
         */
        Class<? extends Throwable> value();
    }
```
필드 설명 - 명사구
요약 설명 - 동사구 (이 애너테이션을 단다는 것이 어떤의미인지 설명)

```java
     /**
      * 이 애너테이션이 달린 메서드는 명시한 예외를 던져야만 성공하는
      * 테스트 메서드임을 나타낸다.
      */
     @Retention(RetentionPolicy.RUNTIME)
     @Target(ElementType.METHOD)
     public @interface ExceptionTest {
         /**
          * 이 애너테이션을 단 테스트 메서드가 성공하려면 던져야 하는 예외.
          * (이 클래스의 하위 타입 예외는 모두 허용된다.)
          */
         Class<? extends Throwable> value();
     }
```

패키지를 설명하는 문서화주석은 `package-info.java`파일에 작성
- 패키지 선언을 반드시 포함
- 패키지 선언 관련 애너테이션 추가로 포함 가능

> 클래스 혹은 정적 메서드가 스레드 안전하든 그렇지 않든, 스레드 안전 수준을 반드시 API 설명에 포함해야한다. 

자바독은 메서드 주석을 상속시킬 수 있다. 

`{@inferitDoc}` : 상위 타입의 문서화 주석 일부 상속
거의 똑같은 문서화 주석 여러 개를 유지보수하는 부담을 줄일 수 있으나, 사용하기 까다롭고 제약도 조금 있다. 

여러 클래스가 상호작용하는 복잡한 API라면 문서화 주석외에도 전체 아키텍처를 설명하는 별도의 설명이 필요하다. 따라서 존재한다면 주석에 링크를 제공해주자. 

자바독은 자바독 문서를 올바르게 작성했는지 확인하는 기능을 제공한다.
부가적으로 아래의 도구를 사용하면 더 완벽하다. 
- 체크스타일 같은 IDE 플러그인
- HTML 파일에 HTML 유효성 검사기 사용

> 정말 잘 쓰인 문서인지를 확인하는 유일한 방법은 **자바독 유틸리티가 생성한 웹페이지를 읽어보는 길뿐이다.** 따라서 다른 사람이 사용할 API라면 반드시 모든 API 요소를 검토하라

