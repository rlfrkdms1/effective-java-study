# 불변 클래스
불변 클래스란 그 인스턴스의 내부 값을 수정할 수 없는 클래스이다. 

불변 클래스는 가변 클래스보다 설계하고 구현하고 사용하기 쉬오며, 오류가 생길 여지도 적고 훨씬 안전하다. 

## 불변 클래스의 다섯가지 규칙

1. 객체의 상태를 변경하는 메서드를 제공하지 않는다. 
    ```java
    public class Point {
        public int x;
        public int y;
    
        public void setX(int x) {
            this.x = x;
        }
    
        public void setY(int y) {
        this.y = y;
        }
    }
    ```
    위의 setter와 같은 변경자를 사용하지 말아야한다. 
    
2. 클래스를 확장할 수 없도록 한다. 
   - 하위 클래스에서 부주의하게 혹은 나쁜 의도로 객체의 상태를 변하게 만드는 사태를 막아준다. 
   ```java
       public final class Point {
        public int x;
        public int y;
       }
    ```
    class를 final로 선언해 상속할 수 없도록 한다. 
    
3. 모든 필드를 final로 선언한다. 
      ```java
       public final class Point {
         public final int x;
         public final int y;
       }
     ```
4. 모든 필드를 private으로 선언한다. 
      ```java
       public final class Point {
         private final int x;
         private final int y;
       }
     ```
5. 자신 외에는 내부의 가변 컴포넌트에 접근할 수 없도록 한다. 
   - 생성자, 접근자, readObject메서드 모두에서 방어적 복사를 수행해라
   ```java
       public final class Point {
         private final int x;
         private final int y;
         private final List<Integer> distances;
         
         public List<Integer> getDistances(){
         	return Collections.unmodifiableList(distances);
         }
       }
   ```
조금 더 긴 예제를 살펴보자. 
   
```java
public final class Complex {
    private final double re;
    private final double im;

    public static final Complex ZERO = new Complex(0, 0);
    public static final Complex ONE  = new Complex(1, 0);
    public static final Complex I    = new Complex(0, 1);

    public Complex(double re, double im) {
        this.re = re;
        this.im = im;
    }

    public double realPart()      { return re; }
    public double imaginaryPart() { return im; }

    public Complex plus(Complex c) {
        return new Complex(re + c.re, im + c.im);
    }

    // 코드 17-2 정적 팩터리(private 생성자와 함께 사용해야 한다.) (110-111쪽)
    public static Complex valueOf(double re, double im) {
        return new Complex(re, im);
    }

    public Complex minus(Complex c) {
        return new Complex(re - c.re, im - c.im);
    }

    public Complex times(Complex c) {
        return new Complex(re * c.re - im * c.im,
                re * c.im + im * c.re);
    }

    public Complex dividedBy(Complex c) {
        double tmp = c.re * c.re + c.im * c.im;
        return new Complex((re * c.re + im * c.im) / tmp,
                (im * c.re - re * c.im) / tmp);
    }

    @Override public boolean equals(Object o) {
        if (o == this)
            return true;
        if (!(o instanceof Complex))
            return false;
        Complex c = (Complex) o;

        // == 대신 compare를 사용하는 이유는 63쪽을 확인하라.
        return Double.compare(c.re, re) == 0
                && Double.compare(c.im, im) == 0;
    }
    @Override public int hashCode() {
        return 31 * Double.hashCode(re) + Double.hashCode(im);
    }

    @Override public String toString() {
        return "(" + re + " + " + im + "i)";
    }
}
```
위의 코드의 사칙연산에서 반환값이 모두 re, im이 아닌 새로운 Complex 인스턴스이다. 

## 불변 객체를 사용하는 이유

### 단순하다. 
생성된 시점의 상태를 파괴될 때까지 그대로 간직한다. 

### 근본적으로 스레드 안전하여 따로 동기화할 필요 없다. 
스레드가 동시에 사용해도 절대 훼손되지 않기때문이다. 

### 안심하고 공유할 수 있다. 
스레드 안전하기 때문이다. 

### 자유롭게 공유할 수 있음은 물론, 불변 객체끼리는 내부 데이터를 공유할 수 있다. 

```java
public class BigInteger extends Number implements Comparable<BigInteger> {
    /**
     * The signum of this BigInteger: -1 for negative, 0 for zero, or
     * 1 for positive.  Note that the BigInteger zero <em>must</em> have
     * a signum of 0.  This is necessary to ensures that there is exactly one
     * representation for each BigInteger value.
     */
    final int signum;

    /**
     * The magnitude of this BigInteger, in <i>big-endian</i> order: the
     * zeroth element of this array is the most-significant int of the
     * magnitude.  The magnitude must be "minimal" in that the most-significant
     * int ({@code mag[0]}) must be non-zero.  This is necessary to
     * ensure that there is exactly one representation for each BigInteger
     * value.  Note that this implies that the BigInteger zero has a
     * zero-length mag array.
     */
    final int[] mag;
    
    public BigInteger negate() {
        return new BigInteger(this.mag, -this.signum);
    }
```
위의 `negate()` 메서드는 크기가 같고 부호만 반대인 새로운 BigInteger를 생성하는데, 이때 배열은 비록 가변이지만 복사하지 않고 원본 인스턴스와 공유해도 된다. => 새로 만든 인스턴스도 원본 인스턴스 내부 배열을 그대로 가리킨다. 

### 객체를 만들 때 다른 불변 객체들을 구성요소로 사용하면 이점이 많다. 
값이 바뀌지 않는 구성요소들로 이뤄진 객체라면, 불변식을 유지하기 훨신 수월하기 때문이다. 

Map의 key와 Set의 원소로 쓰기에 안성 맞춤이다. 예로 `Map<DayOfWeek, Integer>` 처럼 enum으로 불변인 DayOfWeek사용하는 것이다. 만약 DayOfWeek이 가변이었다면 DayOfWeek를 수정해 불변식을 깨뜨릴 수 있다. 

### 불변 객체는 그 자체로 실패 원자성을 제공한다. 

상태가 절대 변하지 않으니 잠깐이라도 불일치 상태에 빠질 가능성이 없다. 

## 단점

### 값이 다르면 반드시 독립된 객체로 만들어야 한다. 
백만 비트 짜리 BigInteger에서 비트 하나를 바꿔야한다고 해보자. 

```java
BigInteger moby = ...;
moby = moby.flipBit(0);
```

flipBit는 새로운 BigInteger 인스턴스를 생성하는데, 원본과 한비트만 다른 백만 비트짜리 인스턴스를 생성하는 것이다. 이는 BigInteger의 크기에 비례해 시간과 공간을 잡아 먹는다. 

### 해결책

#### 다단계 연산들을 예측하여 기본 기능으로 제공
예로 BigInteger는 모듈러 지수 같은 다단계 연산 속도를 높여주는 가변 동반 클래스를 package-private으로 두고 있다. 따라서 클라이언트가 사용하지 않아도 BigInteger가 알아서 사용한다. 

```java
    /**
     * Returns a BigInteger whose value is {@code (this / val)} using an O(n^2) algorithm from Knuth.
     *
     * @param  val value by which this BigInteger is to be divided.
     * @return {@code this / val}
     * @throws ArithmeticException if {@code val} is zero.
     * @see MutableBigInteger#divideKnuth(MutableBigInteger, MutableBigInteger, boolean)
     */
    private BigInteger divideKnuth(BigInteger val) {
        MutableBigInteger q = new MutableBigInteger(),
                          a = new MutableBigInteger(this.mag),
                          b = new MutableBigInteger(val.mag);

        a.divideKnuth(b, q, false);
        return q.toBigInteger(this.signum * val.signum);
    }
```
위의 `new MutableBigInteger()`등의 생성자는 모두 package-private이다. 

```java
    /**
     * The default constructor. An empty MutableBigInteger is created with
     * a one word capacity.
     */
    MutableBigInteger() {
        value = new int[1];
        intLen = 0;
    }
```

## 불변 클래스를 만드는 방법

### 상속하지 못하게 하는 방법
모든 생성자를 private 혹은 package-private으로 만들고 public 정적 팩터리를 제공
```java

public final class Complex {
    private final double re;
    private final double im;


    public Complex(double re, double im) {
        this.re = re;
        this.im = im;
    }

    public static Complex valueOf(double re, double im) {
        return new Complex(re, im);
    }
}
```
package-private 구현 클래스를 원하는 만큼 만들어 활용할 수 있으니 훨씬 유연하다. 

> 다수의 구현 클래스를 활용한 유연성을 제공하고 이에 더해 다음 릴리스에서 객체 캐싱 기능을 추가해 성능을 끌어올릴 수도 있다. 

## 정리

### 클래스는 꼭 필요한 경우가 아니라면 불변이어야 한다. 

getter가 있다해서 무조건 setter를 만들지 말자

### 불변으로 만들 수 없는 클래스라도 변경할 수 있는 부분을 최소한으로 줄이자. 

### 다른 합당한 이유가 없다면 모든 필드는 private final 이어야한다. 
### 생성자는 불변식 설정이 모두 완료된, 초기화가 완벽히 끝난 상태의 객체를 생성해야 한다. 

#### 출처

이펙티브 자바 3/E

