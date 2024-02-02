# 명명 규칙
자바 명명 규칙은 크게 철자와 문법, 두 범주로 나뉜다. 

규칙은 특별한 이유없인 반드시 따라야하며 이를 어긴 API는 사용하기도, 유지보수 하기도 어렵다. 철자 규칙이나 문법 규칙을 어기면 다른 프로그래머들이 코드를 읽기 어렵고, 오해할 수 있으며 오류까지 발생할 수 있다. 
## 철자 규칙

패키지, 클래스, 인터페이스, 메서드, 필드, 타입 변수의 이름을 다룬다. 

### 패키지, 모듈

- 패키지와 모듈 이름은 각 요소를 점으로 구분하여 계층적으로 짓는다. 
- 요소들은 모두 소문자 알파벳 혹은 (드물게) 숫자로 이뤄진다. 
- 조직 바깥에서도 사용될 패키지라면 조직의 인터넷 도메인 이름을 역순으로 사용한다. 
   - ex) edu.cmu, com.google, org.eff
- 예외
   - 표준 라이브러리와 선택적 패키지들은 각각 java와 javax로 시작
- 패키지 이름의 나머지는 해당 패키지를 설명하는 하나 이상의 요소로 이뤄진다. 
   - 각 요소는 일반적으로 8자 이하의 짧은 단어
      - utilities 보다는 util
      - 여러 단어로 구성 -> awt (각 단어의 첫글자만 따서)
      - 보통 한 단어 혹은 약어
- 많은 기능 제공 -> 계층을 나눠 더 많은 요소로 구성
   - java.util은 java.util.concurrent.atomic과 같이 그 밑에 수많은 패키지를 가지고 있다. 
   - 하부의 패키지 : 하위 패키지
### 클래스와 인터페이스 (+열거 타입, 애너테이션)
- 하나 이상의 단어
- 각 단어는 대문자로 시작
   - List, FutherTask
- 여러 단어의 첫 글자만 딴 약자, max, min 처럼 널리 통용되는 줄임말을 제외하고는 단어를 줄여쓰지 않도록 한다. 
   - HttpUrl
### 메서드와 필드
첫 글자를 소문자로 쓴다는 점만 빼면 클래스 명명 규칙과 같다. 
ex ) remove, ensureCapacity

#### 상수 필드
상수 필드는 값이 불변인 static final 필드이며, static final 필드의 타입이 기본 타입이나, 불변 참조 타입이면 상수 필드다. static final 필드이면서 가리키는 객체가 불변이라면 타입은 가변이어도 상수 필드다. 


상수 필드는 구성하는 모든 단어가 대문자여야 한다. 단어사이는 밑줄로 구분한다. 
ex ) VALUES, NEGATIVE_INFINITY


### 지역 변수
다른 멤버와 비슷한 명명 규칙 적용

약어를 사용해도 그 변수가 사용되는 문맥에서 의미를 쉽게 유출할 수 있기 때문에 약어를 사용해도 된다. 
ex ) i, denom, houseNum

입력 매개변수도 지역변수의 하나이지만, 지역변수보단 신경 써야한다. 

### 타입 매개변수
보통 한 문자로 표현

| 문자 | 설명 | 
|-|-|
|T|임의의 타입|
|E|컬렉션 원소의 타입|
|K,V| 맵의 키와 값|
|X|예외|
|R|메서드의 반환 타입|
|T,U,V or T1,T2,T3| 임의 타입 시퀀스|

### 정리

| 식별자 타입 | 예 |
|-|-|
|패키지와 모듈| `org.junit.jupiter.api`, `com.google.common.collect`|
|클래스와 인터페이스|`Stream`, `FutureTask`, `LinkedHashMap`, `HttpClient`|
|메서드와 필드|`remove`, `groupingBy`, `getCrc`|
|상수 필드|`MIN_VALUE`, `NEGATIVE_INFINITY`|
|지역 변수|`i`, `denom`, `houseNum`|
|타입 매개변수|`T`, `E`, `K`, `V`, `X`, `R`, `U`, `V`, `T1`, `T2`|

## 문법 규칙
철자 규칙과 비교하면 더 유연하고 논란도 많다. 

### 패키지
따로 규칙은 없다. 

### 클래스 

- 객체 생성 클래스, 열거타입
    - 보통 단수 명사나, 명사구 사용
    - Thread, PriorityQueue, ChessPiece
- 객체 생성X 클래스
    - 복수형 명사
    - Collectors, Collections

### 인터페이스

클래스와 똑같이 짓거나 (Collection, Comparator), able혹은 ible로 끝나는 형용사로 짓는다. (Runnable, Iterable, Accessible)

### 애너테이션
지배적인 규칙이 없어 명사, 동사, 전치사, 형용사가 두루 쓰인다. 
ex) BindingAnnotation, Inject, ImplementedBy, Singleton

### 메서드

- 어떤 동작을 수행
    - 동사나, (목적어를 포함한) 동사구 
    - append, drawImage
- boolean값을 반환
    - is나 (드물게) has로 시작하고 명사나 명사구, 혹은 형용사로 기능하는 아무 단어나 구로 끝나도록
    - isDigit, isProbablePrice, isEmpty, isEnabled, hasSiblings
- 반환 타입이 boolean이 아니거나, 해당 인스턴스의 속성을 반환하는 메서드
    - 명사, 명사구, 혹은 get으로 시작하는 동사구로 짓는다. 
    - size, hashCode, getTime

get으로 시작하는 형태는 주로 자바빈즈 명세에 뿌리를 두고 있다. 따라서 이러한 명명규칙을 따르는 도구와 어우러지는 코드를 작성한다면 이 규칙 (get-)을 사용해도 된다. getter, setter등

#### 특별한 메서드 이름
- 객체의 타입을 바꿔 다른 타입의 또 다른 객체를 반환하는 인스턴스 메서드
   - toType
   - toString, toArray
- 객체의 내용을 다른 뷰로 보여주는 메서드
   - asType
   - asList
- 객체의 값을 기본 타입 값으로 반환하는 메서드
   - typeValue
   - intValue
- 정적 팩터리
   - from, of, valueOf, instance, getInstance, newInstance, getType, newType
   
### 필드

직접 노출될 일이 거의 없기 때문에 덜 명확하고 덜 중요하다. 

- boolean 타입의 필드
    - boolean 접근자 메서드에서 앞 단어를 뺀 형태
    - initiallized, composite
- 다른 타입의 필드
    - 명사나 명사구
    - height, digits, bodyStyle
 
지역변수 이름도 필드와 비슷하게 지으면 되나, 조금 더 느슨하다. 


#### 출처 

이펙티브 자바 3/E


