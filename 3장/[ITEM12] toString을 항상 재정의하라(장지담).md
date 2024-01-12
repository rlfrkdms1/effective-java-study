# toString을 재정의해야 하는 이유

- Object의 toString은 `Robot@adbbd`처럼 `클래스이름@16진수해시코드`를 반환한다.
- toString은 간결하면서 사람이 읽기 쉬운 형태의 유익한 정보를 반환해야 한다
- 모든 하위 클래스에서 이 메서드를 재정의하자
- println, printf, assert 구문에 넘길 때 등 사용되는 곳이 많다.

# toString 재정의 방법
- 객체가 가진 주요 정보를 모두 반환하는게 좋다. 
  - 객체가 거대하다면 요약 정보를 반환하자.
    
```java
public class Robot {

    private String name;
    private int age;

    public Robot(String name, int age){
        this.name = name;
        this.age = age;
    }

    @Override
    public String toString() {
        return "Robot{" +
                "name='" + name + '\'' +
                '}';
    }
}
```
```java
    @Test
    void test(){
        Robot r1 = new Robot("mong",10);
        Robot r2 = new Robot("mong",20);

        Assertions.assertEquals(r1,r2);
    }
```

![image](https://github.com/rlfrkdms1/effective-java-study/assets/96513365/ac263e17-3359-467e-bd00-b68a3533893b)
- Robot의 name만 toString에 표시되게 정의했다.
- 디버깅하기 힘들다.

- toString을 정의할 때 반환값 포멧을 문서화할지 정해야 한다.
  - 포멧을 명시하던 아니던 의도를 명확히 밝혀야 한다. 
  - 전화번호, 행렬 같은 값 클래스는 문서화하는 것이 좋다.
  - 포맷을 명시하면 포맷에 맞는 문자열과 객체를 상호 변환할 수 있는 정적 팩터리나 생성자를 제공하는 것이 좋다.
  - 포맷을 명시하면 평생 그 포맷에 얽메이게 되고 변경에 취약하다.

```java
    /**
     * 이 로봇의 문자열 표현을 반환한다.
     * 문자열은 "name : 이름, 나이 : 나이" 형태로 구성된다.
     * 이름은 name 필드, 나이는 age 필드다.
     * 
     * 이름이 빈칸이라면 ~~ 
     * 나이가 9자리 이상이면, ~~
     */
```
- 포멧을 명시하려면 정확히 명시해야 한다.
- 포멧에 얽메이게 되고 변경에 취약하다.

```java
    /**
     * 이 로봇의 문자열 표현을 반환한다.
     * 일반적인 형태는 ~~~ 다. 
     * 정해지지 않았으며 추후에 변경될 수 있다
     */
```
- 포멧을 명시하지 않으면 추후에 변경될 수 있다는 것을 명시하자.

- 포맷 명시 여부와 상관없이 toString이 반환한 값에 포함된 정보를 얻어올 수 있는 api를 제공하자.
  - 예를 들어 Robot의 age, battery, power에 대한 접근자를 제공해야 한다. 
  - 제공하지 않는다면 사용자는 toString의 반환값을 파싱해야 한다. 성능이 저하되고 불필요한 작업이다.
```java
    @Override
    public String toString() {
        return String.format("name : %s, age : %s", name, age);
    }
```
- 위와 같이 toString을 재정의했다면 toString을 통해 반환하는 값인 name과 age의 접근 api인 getter를 제공하자.

# toString을 재정의하지 않아도 되는 경우
- 정적 유틸리티 클래스는 객체를 생성하지 않기 때문에 toString을 재정의하지 않아도 된다.
- 열거형은 이미 name을 반환하는 완벽한 toString을 java가 제공하므로 재정의하지 않아도 된다.
- 객체를 생성할 수 없는 추상 클래스더라도 하위 클래스들이 공유해야 할 문자열 표현이 있으면 toString을 재정의한다.



