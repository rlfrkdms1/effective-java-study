# 예외 무시 
```java
        try {
            Robot r = new Robot(1);
            r.charge(-1);
        } catch (IllegalArgumentException e) {
        }

        try {
            Robot r = new Robot(1);
            r.charge(-1);
        } catch (IllegalArgumentException e) {
            e.printStackTrace(); 
        }
```
- catch 블록에서 아무 일도 하지 않거나 로그만 찍는 것을 예외 무시라 한다.
- 예외의 존재 이유가 없어진다. 예외는 적절한 조치를 취하기 위해 있는 것이다.
  - 프로그램이 예외를 내재한 채 동작한다
  - 문제의 원인과 상관없는 곳에서 죽어버릴 수 있다 

# 예외를 무시해야 할 때
```java
    public static void close(FileInputStream fileInputStream) {
        try {
            fileInputStream.close();
        } catch (IOException ignored) {
            // 파일의 상태를 변경하지 않아 복구할 것이 없고, 스트림을 close한다는 것은 필요한 정보를 다 읽었다는 뜻이므로 예외를 무시함
            ignored.printStackTrace(); 
        }
    }
```
- FileInputStream을 close 할 때
  - FileInputStream은 입력 전용 스트림이므로 파일의 상태를 변경하지 않으니 복구할 것이 없다.
  - 스트림을 close한다는 것은 필요한 정보는 다 읽었다는 뜻이다.
- 예외를 무시하기로 했으면 변수명을 ignored로 선언하자.
- 무시한 이유를 블록에 남기자
- 반복적으로 발생한다면 로그를 남겨 확인해보자. `ignored.printStackTrace();`
