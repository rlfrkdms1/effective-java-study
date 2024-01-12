필드만을 모아 놓는 클래스는 데이터 필드에 직접 접근할 수 있으므로 캡슐화의 이점을 제공하지 못한다. 

```java

    class Example{
        public int x;
    }
```
위와 같이 x만을 필드로 갖는 클래스가 있는데 public으로 필드가 선언되어 있다면, 사용자는 Example의 인스턴스만으로 x에 접근할 수 있다. 

```java
        Example example = new Example();
        example.x = 10;
```
위와 같이 말이다. 

이렇게 되면, 다음과 같은 단점이 있다. 

#### API를 수정하지 않고는 내부 표현을 바꿀 수 없다.
하지만 getter를 사용하면 값을 반환할 때 부수적인 작업도 할 수 있다. 

#### 불변식을 보장할 수 없다. 
불변식이란 프로그램이 실행되는 동안, 혹은 정해진 기간 동안 반드시 만족해야하는 조건을 말한다. 위의 예제에서 x는 음수값이 되면 안되는 상황일 때 x가 public이면 x에 음수값이 들어가는 것을 막을 수 없다. 

```java
    class Example{
        public int x; //항상 양수여야함
	}
    
    new Example().x = -10; // 음수 값으로 바꿔도 이를 검증할 방법이 없음
```

#### 외부에서 필드에 접근할 때 부수 작업을 수행할 수 없다. 


## 철저한 객체 지향 프로그래머

### 필드는 private, getter와 setter 추가
따라서 필드를 모두 private으로 바꾸고 getter를 추가해 getter를 통해 접근할 수 있도록 해야한다. 

```java
    class Example{
        private int x;

        public int getX() {
            return x;
        }
        
        public int setX(int x) {
            this.x=x;
        }
    }
```

> 패키지 바깥에서 접근할 수 있는 클래스라면 접근자를 제공함으로써 클래스 내부 표현 방식을 언제든 바꿀 수 있는 유연성을 얻을 수 있다. 

### private 중첩 클래스, package-private 클래스
하지만 package-private 클래스 혹은 private 중첩 클래스라면 데이터 필드를 노출한다 해도 하등의 문제가 없다.


```java
public class Outside{
    
    public void test() {
        new Inside().inside = 10;
    }

    static class Inside{
        public int inside;
    }
}
```

```java
public class Outside{

    public void test() {
        new Inside().inside = 10;
    }

    private class Inside{
        public int inside;
    }
}
```
위의 두 예제를 사용한다면, 패키지 외부에선 Inside의 필드가 public이더라도  접근할 수 없다. 또한 Outside에선 Inside의 필드를 조작할 수 있다. 하지만 public 중첩 클래스라면 외부에서 접근이 가능하다. 

이러한 방법이 클래스 선언 면에서나 이를 사용하는 클라이언트 코드 면에서나 접근자 방식보다 훨씬 깔끔하다. 

java의 라이브러리 중 본 규칙을 어기는 대표적인 예가 `java.awt.package`의 `Point`와 `Dimension` 클래스이며 이를 흉내내지 말아야한다. 

## 불변이라면?

직접 노출할 때의 단점이 조금 줄어들지만, 좋은 생각이 아니다. 

API를 변경하지 않고는 표현 방식을 바꿀 수 없고, 필드를 읽을 때 부수 작업을 수행할 수 없다는 단점은 여전하다. 

하지만 불변식은 보장할 수 있다. 

```java
public final class Time {
    private static final int HOURS_PER_DAY    = 24;
    private static final int MINUTES_PER_HOUR = 60;

    public final int hour;
    public final int minute;

    public Time(int hour, int minute) {
        if (hour < 0 || hour >= HOURS_PER_DAY)
            throw new IllegalArgumentException("Hour: " + hour);
        if (minute < 0 || minute >= MINUTES_PER_HOUR)
            throw new IllegalArgumentException("Min: " + minute);
        this.hour = hour;
        this.minute = minute;
    }

    // 나머지 코드 생략
}
```
위의 코드를 살펴보면 `hour, minute`이 public 이지만 불변으로 선언되어있다. 이는 생성자를 통해 불변식을 보장하는 것을 볼 수 있다. 

#### 출처

이펙티브 자바 3/E


