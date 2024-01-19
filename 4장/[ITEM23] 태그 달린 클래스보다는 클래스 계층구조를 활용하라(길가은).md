태그 값으로 알려주는 클래스를 살펴보자. 

```java
class Figure {
    enum Shape { RECTANGLE, CIRCLE };

    // 태그 필드 - 현재 모양을 나타낸다.
    final Shape shape;

    // 다음 필드들은 모양이 사각형(RECTANGLE)일 때만 쓰인다.
    double length;
    double width;

    // 다음 필드는 모양이 원(CIRCLE)일 때만 쓰인다.
    double radius;

    // 원용 생성자
    Figure(double radius) {
        shape = Shape.CIRCLE;
        this.radius = radius;
    }

    // 사각형용 생성자
    Figure(double length, double width) {
        shape = Shape.RECTANGLE;
        this.length = length;
        this.width = width;
    }

    double area() {
        switch(shape) {
            case RECTANGLE:
                return length * width;
            case CIRCLE:
                return Math.PI * (radius * radius);
            default:
                throw new AssertionError(shape);
        }
    }
}
```

단점을 살펴보자. 

- 쓸데 없는 코드가 많다. 
   - 열거 타입 선언, 태그 필드, switch문 등
- 가독성이 나쁘다. 
   - 여러 구현이 한 클래스에 혼합됨
- 메모리를 많이 사용함
   - 다른 의미를 위한 코드도 언제나 함께하기 때문 
   ```java
       double area() {
        switch(shape) {
            case RECTANGLE:
                return length * width;
            case CIRCLE:
                return Math.PI * (radius * radius);
            default:
                throw new AssertionError(shape);
        }
    }
    ```
- 불필요한 초기화 코드
   - 필드들을 final로 선언하려면 해당 의미에 쓰이지 않는 필드들까지 생성자에서 초기화해야함
   ```java

    // 다음 필드들은 모양이 사각형(RECTANGLE)일 때만 쓰인다.
    final double length;
    final double width;

    // 다음 필드는 모양이 원(CIRCLE)일 때만 쓰인다.
    double radius;

    // 원용 생성자
    Figure(double radius) {
        shape = Shape.CIRCLE;
        this.radius = radius;
        length = 0;
        width = 0;
    }
    ```
    length와 width는 원일때 쓰이지 않음에도 final이므로 원용 생성자를 쓸때도 초기화를 해줘야한다. 따라서 낭비다. 

따라서, 태그 달린 클래스는 장황하고, 오류를 내기 쉽고, 비효율적이다. 

### 클래스 계층 구조로 바꿔보자.
1. root가 될 추상 클래스를 정의한다.
	```java
	public abstract class Figure {
	   
	}
    ```
2. 태그 값에 따라 동작이 달라지는 메서드들을 루트 클래스의 추상메서드로 선언한다. 
	```java
	public abstract class Figure {		
	    abstract double area();
	}
    ```
3. 태그 값에 상관없이 동작이 일정한 메서드들을 루트 클래스에 일반 메서드로 추가한다. 
	```java
	public abstract class Figure {	    
    
	    final String getColor() {
	        return color;
	    }
		
	    abstract double area();
	}
    ```
4. 모든 하위 클래스에서 공통으로 사용하는 데이터 필드들도 전부 루트 클래스로 올린다. 
	```java
	public abstract class Figure {
	    final String color;
	    
	    public Figure(String color) {
	        this.color = color;
	    }
    
	    final String getColor() {
	        return color;
	    }
		
	    abstract double area();
	}
    ```
5. 루트 클래스를 확장한 구체 클래스를 의미별로 하나씩 정의한다. 
    ```java
    class Circle extends Figure {

	}

	class Rectangle extends Figure {
    
	}
	```
   
6. 각 하위 클래스에는 각자의 의미에 해당하는 데이터 필드들을 넣는다. 
   ```java
   public final class Circle extends Figure {
	    final double radius;
	}

	public class Rectangle extends Figure {
	    final double width;
	    final double length;
	}
	```
7. 루트 클래스가 정의한 추상 메서드를 각자의 의미에 맞게 구현한다. 

	```java
	public abstract class Figure {
	    final String color;
	    
	    public Figure(String color) {
	        this.color = color;
	    }
    
	    final String getColor() {
	        return color;
	    }
		
	    abstract double area();
	}
    class Circle extends Figure {

	    final double radius;
    
	    Circle(String color, double radius) {
	        super(color);
	        this.radius = radius;
	    }
    
	    @Override
	    double area() {
	        return Math.PI * (radius * radius);
	    }
	}

	class Rectangle extends Figure {

	    final double width;
	    final double length;
    
	    Rectangle(String color, double width, double length) {
	        super(color);
	        this.width = width;
	        this.length = length;
 	   }
    
	    @Override
	    double area() {
	        return width * length;
	    }
	}
	```
    
독립적으로 계층구조를 확장하고 함께 사용할 수 있다. 타입이 의미별로 따로 존재해 변수의 의미를 명시하거나 제한할 수 있고, 특정 의미만 매개변수로 받을 수 있다. 

타입사이의 자연스러운 계층 관계를 반영할 수 있어 유연성은 물론 컴파일 타입 타입 검사 능력을 높여준다. 

#### 출처 
이펙티브 자바 3/E
