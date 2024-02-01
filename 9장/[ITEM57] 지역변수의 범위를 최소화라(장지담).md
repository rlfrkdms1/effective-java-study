지역변수 범위를 최소로 줄이면 코드 가독성과 유지보수성이 높아지고 오류 가능성은 낮아진다

### 가장 처음 쓰일 때 선언하기 
- 미리 선언해두면 코드가 어수선해지고 가독성 떨어짐
- 지역변수의 범위 : 선언 지점 ~ 블록 끝까지

### 선언과 동시에 초기화하기 
- 초기화에 필요한 정보가 충분하지 않으면 충분해질 때 까지 미루기
- try-catch는 예외

```java
    public static void main(String[] args) {
        Robot r = null;
        try {
            // 초기화 과정에서 예외 발생 가능 
            r = new Robot(1);
        } catch (Exception e) {
            System.out.println(e);
        }
        // 변수를 try 바깥에서도 사용해야 함
        System.out.println(r);
    }
```
- 초기화 과정에서 예외가 발생할 수 있는 경우 try 블록에서 초기화한다. 그렇지 않으면 예외가 전파된다
- 그리고 변수를 try 밖에서도 사용해야 한다면 try 앞에서 선언한다

### while보다 for 사용하기
- for나 foreach는 반복변수의 범위가 for 안으로 제한된다
```java
        List<Robot> list = new ArrayList<>();

        for (Robot robot : list) {
            System.out.println(robot.age);
        }

        // foreach에서 iterator를 사용할 수 없다 - 컴파일에러
        for (Robot robot : list.iterator()) {
            
        }
        
        for (Iterator<Robot> it = list.iterator(); it.hasNext();){
            
        }
```
- iterator를 사용해야 할 때는 foreach대신 for를 사용하자.

```java
        Iterator<Robot> i1 = list.iterator();
        while(i1.hasNext()){
            
        }

        Iterator<Robot> i2 = list.iterator();
        while(i1.hasNext()){

        }
```
- while문은 복붙 실수가 발생할 수 있다. i1의 범위가 끝나지 않아 두번째 while에서 i1을 그대로 사용해도 컴파일에러로 잡을 수 없다.

```java
        for (Iterator<Robot> i = list.iterator(); i.hasNext();){
            
        }

        for (Iterator<Robot> i2 = list.iterator(); i.hasNext();){

        }

        // 사실 변수명을 다르게 할 필요조차 없다.
        for (Iterator<Robot> i = list.iterator(); i.hasNext();){

        }
```
- for문은 복붙 실수를 방지해준다.
- 사실 변수명을 i2로 다르게 지을 필요조차 없다. i의 범위가 이전 for문에서 끝나서 같은 변수명 i를 또 쓸수 있다.
- for문 내에서 여러개 변수를 선언할 수 있어 효율적이다. 반면 while문은 변수를 따로 선언해야 한다.

### 메서드를 작게 유지하고 한 가지 기능에 집중하자
- 메서드에서 기능 A, B를 처리한다면 A기능을 위한 지역변수도 B기능을 처리하는 코드에서 접근할 수 있다.
- A, B기능을 위한 메서드를 분리하자.
