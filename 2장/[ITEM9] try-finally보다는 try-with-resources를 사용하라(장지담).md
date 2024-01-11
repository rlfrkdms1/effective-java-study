# try-catch

java에는 close를 호출해 닫아줘야 하는 자원이 많다. 클라이언트가 close 호출을 깜빡하기 쉽다. 전통적으로 close 호출을 보장하기 위해 try-catch 구문이 사용되었다.


### 코드가 지저분해진다 
```java
    public static void main(String[] args) throws IOException {
        Resource resource = new Resource();
        try {
            resource.error();
        } finally {
            resource.close();
        }
    }
```
이렇게 close 호출을 보장할 수 있다.
```java
public static void main(String[] args) throws IOException {
        Resource resource = new Resource();
        try {
            resource.error();
            Mineral mineral = new Mineral();
            try {
                mineral.error();
            } finally {
                mineral.close();
            }
        } finally {
            resource.close();
        }
    }
```
하지만 자원 2개만 사용해도 코드가 지저분해진다. 

### 디버깅이 힘들다
```java
@NoArgsConstructor
public class Resource implements Closeable {

    public void error(){
        throw new Exception1();
    }

    @Override
    public void close() throws IOException {
        throw new Exception2();
    }
}
```
Resource의 error와 close 메서드 모두에서 예외를 발생시킨다.
```java
    public static void main(String[] args) throws IOException {
        Resource resource = new Resource();
        try {
            resource.error();
        } finally {
            resource.close();
        }
    }
```
이 코드를 실행해보면 아래와 같은 에러를 확인할 수 있다.
![image](https://github.com/rlfrkdms1/effective-java-study/assets/96513365/8d4fbf7f-c369-447c-b4e8-1c2988722da3)
- 두번째 예외만 표시되어 디버깅을 어렵게 한다

# try-with-resources
try-with-resource는 try-catch의 문제를 해결한다.
ITEM8에서 설명했듯이, **닫아야 하는 자원 클래스가 AutoCloseable을 구현하도록 하고 try-with-resource로 생성하면 close를 자동으로 호출한다.**

### 깔끔하다 
```java
        try (Resource resource = new Resource()) {
            resource.error();
        }
```
```java
        try (Resource resource = new Resource(); Mineral mineral = new Mineral()) {
            resource.error();
            mineral.error();
        }
```
- 자원이 1개인 경우, 2개인 경우 모두 코드가 깔끔하다.

### 디버깅이 쉽다
![image](https://github.com/rlfrkdms1/effective-java-study/assets/96513365/e32350de-386c-4d4a-b4c5-743735477476)
- 최초 발생한 예외가 표시된다.
- 나중에 발생한 예외도 suppressed(숨겨짐)에 표시된다.

---

try-with-resources에서도 catch문을 사용할 수 있다.
```java
        try (Resource resource = new Resource()) {
            resource.error();
        } catch (Exception1 e){
            System.out.println("e = " + e);
        } catch (Exception2 e){
            System.out.println("e = " + e);
        }
```






