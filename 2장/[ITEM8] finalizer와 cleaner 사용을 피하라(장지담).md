# finalizer와 cleaner
java는 객체 소멸자 finalizer와 cleaner를 제공한다. 일반적으로 예측할 수 없고, 불필요하다.
- 수행 시점을 보장하지 않는다. 따라서 제때 실행되어야 하는 작업을 할 수 없다.
- 수행 여부를 보장하지 않는다. 상태를 영구적으로 수정하는 작업을 할 수 없다.
- finalizer 동작 중 발생한 예외는 무시되고, 처리할 작업이 남아도 그 순간 종료된다. (cleaner에서는 이러한 문제가 발생하지 않는다)
- 성능이 떨어진다
- finalizer는 finalizer 공격에 노출되어 보안 문제를 일으킬 수 있다
  - 생성자나 직렬화 과정에서 예외가 발생하면 이 생성되다 만 객체의 하위 클래스에서 악의적인 finalizer가 실행될 수 있다. 이 finalizer는 정적 필드에 자신의 참조를 할당해 GC가 수집하지 못하게 막을 수 있다.
```java
public class Robot {
    static Robot robot;

    int life;

    public Robot(int life) {
        if (life <= 0) {
            throw new IllegalArgumentException("negative life value");
        }
        this.life = life;
    }

    public void finalize() {
        robot = this;
    }
}
```
- finalizer에서 java 코드를 실행할 수 있다. 정적 필드에 자신의 참조를 할당하는 코드를 실행할 수도 있다. 참조가 생겼으니 GC가 수집하지 못한다.
- 생성자에서 값 검증에 실패해도 객체를 생성할 수 있다.
- finalizer가 있는 객체는 final 클래스로 만들어 하위 클래스를 만들 수 없도록 하자.
- final 클래스가 아니라면 아무 일도 하지 않는 finalize 메서드를 만드고 final로 선언하자.

# AutoCloseable
- finalizer와 cleaner의 대체로 파일, 스레드 등 종료해야 할 자원을 담고 있는 클래스에서 AutoCloseable을 구현하고, 클라이언트에서 인스턴스를 다 쓰고 나면 close 메서드를 호출하도록 하자.
- 각 인스턴스는 자신이 닫혔는지 추적하는 것이 좋다
    - close 메서드에서 자신이 닫혔음 필드에 기록하고, 다른 메서드에서 이 필드를 검사해서 자원이 닫힌 뒤 사용하려 했다면  IllegalStateException을 던진다.



# finalizer와 cleaner의 쓰임새
### 자원이 소유자가 close 메서드를 호출하지 않는 것에 대한 안전망
finalizer나 cleaner는 수행 시간, 수행 여부를 보장하지는 않지만, 그래도 안하는 것보다는 낫다. 

```java
public class Robot implements AutoCloseable {

    private static final Cleaner cleaner = Cleaner.create();

    private static class Garbage implements Runnable {
        int junkFiles;

        Garbage(int junkFiles) {
            this.junkFiles = junkFiles;
        }

        @Override
        public void run() {
            System.out.println("청소");
            junkFiles = 0;
        }
    }

    private final Garbage garbage;

    private final Cleaner.Cleanable cleanable;

    public Robot(int junkFiles) {
        garbage = new Garbage(junkFiles);
        cleanable = cleaner.register(this, garbage);
    }

    @Override
    public void close() throws Exception {
        cleanable.clean();
    }
}
```
위 예시에서는 Runnable을 구현한 Garbage가 닫아야 할 자원이다. 

동작을 이해하기 위해 일부 코드를 살펴보자.
```java
public interface AutoCloseable {
    /**
     * Closes this resource, relinquishing any underlying resources.
     * This method is invoked automatically on objects managed by the
     * {@code try}-with-resources statement.
~~~ 이하생략 ~~~~~
    void close() throws Exception;
}
```
AutoCloseable 인터페이스다. 
close()에 관한 설명 중 일부는 아래와 같다. 
> 이 리소스를 닫고 기본 리소스를 모두 취소합니다. 이 메서드는 try-with-resources 문으로 관리되는 개체에서 자동으로 호출됩니다.

- Runnable을 구현한 객체는 try-with-resources 문으로 생성하면 `close()` 를 자동 호출한다. 

```java
    public Cleanable register(Object obj, Runnable action) {
        Objects.requireNonNull(obj, "obj");
        Objects.requireNonNull(action, "action");
        return new CleanerImpl.PhantomCleanableRef(obj, this, action);
    }
```
Cleaner의 register 메서드
- Cleanable 구현체 PhantomCleanableRef를 생성해서 반환한다. 
```java
        public PhantomCleanableRef(Object obj, Cleaner cleaner, Runnable action) {
            super(obj, cleaner);
            this.action = action;
        }
```
PhantomCleanableRef의 생성자다. action을 초기화한다. 
```java
@Override
    public final void clean() {
        if (remove()) {
            super.clear();
            performCleanup();
        }
    }
```
PhantomCleanableRef가 상속하는 PhantomCleanable<Object>의 clean 메서드다.
performCleanup을 호출한다. 
```java
@Override
        protected void performCleanup() {
            action.run();
        }
```
PhantomCleanableRef의 performCleanUp 메서드로, Runnable 타입인 action의 run을 호출한다.
즉 Cleanable의 clean을 호출하면 Cleanable을 생성할 때 호출한 `cleaner.register(this, garbage);`에 전달한 Runnable의 구현체 garbage의 run 메서드가 실행된다. 

- garbage의 run이 호출되는 2가지 경우
  - 1. 클라이언트가 Robot의 `close()` 호출 -> `clean()` 호출 -> `run()` 호출
        1) 클라이언트가 직접 Robot의 `close()` 호출
        2) Robot을 try-catch-resource문으로 생성하면 `close()`는 자동 호출
  - 2. GC가 Robot을 회수할 때 까지 클라이언트가 Robot `close()`를 호출하지 않으면 cleaner가 Garbage의 `run()` 호출
       - finalizer와 cleaner의 쓰임새인 안전망 역할
       - 시간 보장 X
       - 여부 보장 X

클라이언트가 자원을 try-catch-resource문으로 생성했거나, `close()`를 직접 호출하지 않은 경우, 안전망 역할의 cleaner가 작동한다. 이 경우, 청소가 될 수도, 안될 수도 있다. 
         
### native peer와 연결된 객체
> native peer: java가 아닌 c, cpp같은 다른 프로그래밍 언어
- 일반 java 객체가 아니므로 GC가 수거하지 못한다.
- 성능 저하를 감당할 수 있고, 네이티브 피어가 심각한 자원을 갖고 있지 않은 경우 사
  - 아닌 경우 `close()` 메서드 사용



    
