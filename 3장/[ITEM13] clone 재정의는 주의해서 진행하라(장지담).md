# Cloneable 인터페이스란?
> Cloneable은 복제해도 되는 클래스임을 나타내는 믹스인 인터페이스(ITEM20)이다.
- clone 메서드는 Cloneable이 아닌 Object에 protected로 선언되어 있다
- Cloneable을 구현하는 것 만으로는 객체 외부에서 clone을 호출할 수 없다.
```java
// 클래스는 Cloneable 인터페이스를 구현하여 해당 메서드가 해당 클래스 인스턴스의 필드 간 복사본을 만드는 것이 적법함을 Object.clone() 메서드에 나타냅니다.
public interface Cloneable {
}
```
- Cloneable 인터페이스에는 아무런 메서드가 없다.
- 이 인터페이스는 Object의 protected 메서드인 clone의 동작 방식을 결정한다
-  Clonealbe을 구현한 클래스의 인스턴스에서 clone을 호출하면 객체의 필드를 하나씩 복사한 객체를 반환한다
-  Cloneable을 구현하지 않은 인스턴스에서 clone을 호출하면 CloneNotSupportedException을 던진다.
-  인터페이스 선언은 보통 해당 클래스가 인터페이스에서 정의한 기능을 사용한다는 뜻인데, Cloneable은 상위 클래스에 정의된 clone의 동작방식을 변경하는 이례적인 경우다

# clone 메서드 일반 규약 
- 객체의 복사본을 생성해 반환한다. 복사의 의미는 클래스에 따라 다를 수 있다
- 일반적인 의미는 이렇다. 아래 규약들은 필수로 지켜야 하는 것은 아니다.
  - x.clone () != x 가 참이다. (복사본과 원본이 일치하지 않는다)
  - x.clone().getClass() == x.getClass() 가 참이다. (복사본과 원본의 인스턴스 타입은 같다)
  - x.clone.equals(x) 가 참이다.(복사본과 원본은 논리적으로 동치이다)
- 관례상 clone이 반환하는 객체는 super.clone()을 통해 얻어야 한다.
  -  Object를 제외한 이 클래스의 모든 상위 클래스가 이 관례를 따른다면 x.clone().getClass() == x.getClass() 가 참이다.
  -  clone이 super.clone을 반환하지 않게 정의할 수도 있다. 하지만 이 클래스를 상속한 클래스의 super.clone이 잘못된 클래스의 객체를 만들 것이고, 하위 클래스의 clone이 오작동할것이다.
- 관례상 반환된 객체와 원본 객체는 독립적이어야 한다.

# clone 메서드 작성 방법
**제대로 작동하는 clone 메서드를 가진 상위 클래스**를 상속해 Cloneable을 구현한다고 하자.

## 가변 상태를 참조하지 않는 클래스용 clone 메서드
```java
    @Override
    protected Robot clone() {
        try {
            return (Robot) super.clone();

        } catch (CloneNotSupportedException e) {
            throw new AssertionError();
        }
    }
```
- 모든 필드가 기본 타입이거나 불변 객체를 참조하는 경우의 clone이다.
- 이 메서드가 동작하게 하려면 Robot이 Cloneable을 구현하게 해야 한다.
- Object의 clone은 Object를 반환하고 이것을 재정의 한 Robot의 clone은 Robot을 반환한다.(공변 반환 타이핑) 클라이언트가 형변환하지 않아도 된다
- try-catch를 사용하는 이유는 Object의 clone이 CloneNotSupportedException을 던지기 때문이다. 

## 가변 상태를 참조하는 않는 클래스용 clone 메서드
가변 객체를 참조하는 필드가 있는 클래스에서 super.clone을 그대로 반환하면 원본과 복제된 객체가 같은 가변 객체를 참조할 것이다. 따라서 원본이나 복제본 중 하나를 수정하면 다른 하나도 수정된다. 불변식을 해치게 된다. 
> 불변식 : 어떤 객체가 정상적으로 동작하기 위해 허물어지지 않아야 하는 값, 식, 상태의 일관성을 보장하기 위해 항상 참이 되는 조건

clone 메서드는 원본 객체에 아무런 해를 끼치지 않는 동시에 복제된 객체의 불변식을 보장해야 한다.

### clone의 재귀적 호출

```java
    @Override
    public Robot clone() {
        try {
            Robot result = (Robot) super.clone();
            result.arm = (Arm)arm.clone();
            return result;
        } catch (CloneNotSupportedException e) {
            throw new AssertionError();
        }
    }
```
- 가변 필드의 clone을 재귀적으로 호출해 이 문제를 해결한다.
- 가변 필드가 배열이라면 형변환하지 않아도 된다. 배열의 clone은 원본 배열과 똑같은 배열을 반환하기 때문이다.
- 위 방법은 가변 객체를 참조하는 필드가 final이라면 사용할 수 없다(`private final Arm arm`).
  - Cloneable 아키텍쳐는 가변 객체를 참조하는 필드는 final로 선언하라는 일반 용법과 충돌한다.
  - 필드가 final로 선언되어 있다면필요에 따라 제거해야 할 수 있다.

### 복잡한 가변 객체
복잡한 가변 객체의 경우, 단순 재귀 호출만으로 문제를 해결할 수 없는 상황이 있다. 
#### 깊은복사 
```java
public class Robot implements Cloneable{
    private int age;
    private Arm[] arms;

    private static class Arm {
        Arm next;

        Arm(Arm next) {
            this.next = next;
        }
    }

    @Override
    protected Object clone() throws CloneNotSupportedException {
        Robot result = (Robot) super.clone();
        result.arms = arms.clone();
        return result;
    }
}
```
- Robot은 Arm 배열 arms의 참조를 갖고 각 Arm은 다음 Arm에 대한 참조를 갖는다. (arm 배열의 각 엔트리가 Arm의 LinkedList인 상황이다)
- clone 메서드에서 가변 배열 arms에 대한 clone의 재귀 호출한다.
![image](https://github.com/rlfrkdms1/effective-java-study/assets/96513365/dd667106-6e83-44a1-8d06-f3541e8c0804)
- 복제본은 자신만의 arms 배열을 갖지만, 원본의 arms 배열과 복제본의 arms 배열이 가리키는 Arm 객체들은 같게 된다.

```java
public class Robot implements Cloneable {
    private int age;
    private Arm[] arms;

    private static class Arm {
        Arm next;

        Arm(Arm next) {
            this.next = next;
        }

        Arm deepCopy() {
            return new Arm(next == null ? null : next.deepCopy());
        }
    }

    @Override
    protected Object clone() throws CloneNotSupportedException {
        Robot result = (Robot) super.clone();
        result.arms = new Arm[arms.length];
        for (int i=0; i<arms.length; i++){
            if (arms[i]!=null){
                result.arms[i] = arms[i].deepCopy();
            }
        }
        return result;
    }
}
```
- Arm에 deepCopy 메서드를 정의한다. 자신이 가리키는 연결리스트를 복사하기위해 자신의 deepCopy를 재귀호출한다.
- Robot의 clone에서는 arms의 각 엔트리의 Arm을 deepCopy로 복사한다.
- 간단한 방법이지만, 연결 리스트를 복사하면 스택오버플러우가 발생할 수 있다.

```java
        Arm deepCopy() {
            Arm result = new Arm(next);
            for (Arm arm = result; arm.next != null; arm = arm.next) {
                arm.next = new Arm(arm.next.next);
            }
            return result;
        }
```
- 재귀 호출이 아닌 반복자를 활용하도록 Arm의 deepCopy를 정의할 수 있다. 

#### 고수준 api를 활용한 복제
- super.clone으로 얻은 객체의 모든 필드를 초기 상태로 설정하고 고수준 api를 활용해 복제할 수 있다.
- 코드가 간단하지만 저수준의 직접 처리보다는 느리다.

# 주의점
- 생성자에서는 재정의 될 수 있는 메서드를 호출하지 않아야 한다.(ITEM19) clone도 마찬가지다.
  - 상위클래스의 clone에서 하위 클래스에서 재정의 한 메서드를 호출하면 하위 클래스에서 호출한 super.clone에서는 하위 클래스에서 재정의 한 메서드가 아닌 상위 클래스의 메서드에 따라 복제되고 하위 클래스는 자신의 상태를 교정할 기회를 잃는다.
  - clone에서 호출하는 고수준 api는 final이거나(재정의하지 못하므로) private이어야 한다(하위 클래스에서 호출하지 못하므로).
- Object의 clone은 CloneNotSupportedException을 던진다. clone을 재정의 한 메서드에서는 throws를 없애고 try-catch로 잡아주는 편이 사용하기 편하다.
- 상속해서 쓰기 위한 클래스(상속용 클래스)에서는 Cloneable을 구현하면 안된다
  - Object가 한 것처럼 작동하는 clone을 protected로 선언하면 Cloneable 구현 여부를 하위 클래스에서 선택할 수 있다.
  - 하위클래스에서 Cloneable을 구현하면 clone에서 super.clone을 호출하고, 이 때 상위 클래스의 protected로 선언한, 작동하는 clone을 호출한다.
```java
    @Override
    protected final Object clone() throws CloneNotSupportedException {
        throw new CloneNotSupportedException();
    }
```
  - clone을 동작하지 않게 구현하고 하위클래스에서 재정의하지 못하게 할 수도 있다.
- Cloneable을 구현한 스레드 안전 클래스를 작성할 때는 clone 메서드 역시 동기화해줘야 한다. Object의 clone은 메서드 동기화를 신경쓰지 않았다.
> Java에서 동기화는 공유 리소스에 대한 여러 스레드의 액세스를 제어하는 ​​기능을 의미한다.

> 스레드 안전(Thread-Satety)란 멀티 스레드 프로그래밍에서 일반적으로 어떤 함수나 변수, 혹은 객체가 여러 스레드로부터 동시에 접근이 이루어져도 프로그램의 실행에 문제가 없는 것을 말한다.
  
# 복사 생성자, 복사 팩터리
```java
    public Robot(Robot robot) {

    }

    public static Robot newInstance(Robot robot) {
        
    }
```
> 복사 생성자 : 자신과 같은 클래스의 객체를 인수로 받는 생성자

> 복사 팩터리 : 복사 생성자를 모방한 정적 팩터리

- 생성자를 쓰지 않고 객체를 생성하는 위험한 방식을 사용하지 않는다
- final 필드 용법과 충돌하지 않는다
- 불필요한 검사 예외를 던지지 않는다
- 형변환이 불필요하다
- 해당 클래스가 구현한 인터페이스를 인수로 받을 수 있다.
    - 원본의 구현 타입에 얽매이지 않고 복사본의 구현 타입을 선택할 수 있다. 
```java
        HashSet<Integer> hs = new HashSet<>();
        hs.add(1);
        hs.add(2);
        TreeSet<Integer> ts = new TreeSet<>(hs);`
```
- TreeSet이 구현한 인터페이스 AbstractSet의 또다른 구현체 HashSet을 복사 생성자의 인자로 받는다.

# 결론 
Cloneable을 이미 구현한 클래스를 확장하는게 아니라면 복사 생성자, 복사 팩터리를 사용할 수 있다.
Cloneable보다 백배 천배 낫다. 




