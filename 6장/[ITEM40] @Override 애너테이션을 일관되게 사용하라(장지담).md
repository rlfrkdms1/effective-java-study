```java
public class Robot {
    private final int id;

    public Robot(int id) {
        this.id = id;
    }

    public int hashCode() {
        return Objects.hashCode(id);
    }

    public boolean equals(Robot robot) {
        return this.id == robot.id;
    }

    public static void main(String[] args) {
        Set<Robot> set = new HashSet<>();
        for (int i = 0; i < 10; i++) {
            set.add(new Robot(1));
        }
        System.out.println(set.size());
    }
}
```
- 출력 결과로 1을 예상하지만 10을 얻는다. 이것은 equals가 재정의가 아닌 다중정의가 되었기 때문이다.
- `@Overide` 애너테이션을 달아 이런 버그를 예방할 수 있다.

![image](https://github.com/rlfrkdms1/effective-java-study/assets/96513365/be8f5bbe-25bc-4fbb-a65c-5f6ef557d769)
- 컴파일타임에 실수를 확인할 수 있다.

```java
    @Override
    public boolean equals(Object o) {
        if (!(o instanceof Robot)) {
            return false;
        }
        Robot r = (Robot) o;
        return id == r.id;
    }
```
- 메서드를 재정의한다는 의도를 명시해 컴파일러가 버그를 찾을 수 있게 한다 .

### 상위 클래스의 메서드를 재정의하는 모든 메서드에 @Override를 달자 
- 예외적으로 구체클레스에서 추상메서드를 재정의 할 때는 달지 않아도 된다. 추상메서드를 재정의하지 않으면 컴파일에러가 발생하기 때문이다. 하지만 달아서 손해볼 것도 없다.
