우리는 대개 문자열을 연결할 때 `+` 기호를 사용한다. 하지만 한 줄 짜리 출력값 혹은 작고 크기가 고정된 객체의 문자열 표현을 만들때는 괜찮으나, 이렇게 않을 경우 성능 저하를 일으킬 수 있다. 

> 문자열 연결 연산자로 문자열 n개를 잇는 시간은 $$n^2$$에 비례한다. 

문자열은 불변이므로 연결할 경우 양쪽의 내용을 모두 복사해야하기 때문이다. 

```java
    public static String statementByPlus() {
        String result = "";
        List<String> target = List.of("effective Java", " ", "Chapter9 ", " ", "item63 ", " ", "문자열 연결은 느리니 주의하라");
        for (int i = 0; i < target.size(); i++) {
            result += target.get(i);
        }
        return result;
    }
```
위의 경우 List의 값이 많아질 수록 메서드는 느려진다. 따라서 성능을 포기 하고 싶지 않다면 String 대신 **`StringBuilder`**를 사용하자. 

```java
    public static String statementByBuilder() {
        List<String> target = List.of("effective Java", " ", "Chapter9 ", " ", "item63 ", " ", "문자열 연결은 느리니 주의하라");
        StringBuilder stringBuilder = new StringBuilder();
        for (int i = 0; i < target.size(); i++) {
            stringBuilder.append(target.get(i));
        }
        return stringBuilder.toString();
    }
```

String을 사용한 메서드의 수행시간은 품ㅁ목 수의 제곱에 비례해 늘어나고, StringBuilder를 사용한 메서드는 선형으로 늘어나므로 연결해야할 문자열이 많을 수록 성능격차가 심해진다. 시간을 재보면, 아래와 같은 결과를 얻을 수 있다. 

```java
    public static String statementByPlus() {
        long beforeTime = System.currentTimeMillis();
        String result = "";
        List<String> target = List.of("effective Java", " ", "Chapter9 ", " ", "item63 ", " ", "문자열 연결은 느리니 주의하라");
        for (int i = 0; i < target.size(); i++) {
            result += target.get(i);
        }
        long afterTime = System.currentTimeMillis();
        long timeDifference = (afterTime - beforeTime);
        System.out.printf("+ 사용할 때 소요 시간 : %d\n", timeDifference);
        return result;
    }

    public static String statementByBuilder() {
        long beforeTime = System.currentTimeMillis();
        List<String> target = List.of("effective Java", " ", "Chapter9 ", " ", "item63 ", " ", "문자열 연결은 느리니 주의하라");
        StringBuilder stringBuilder = new StringBuilder();
        for (int i = 0; i < target.size(); i++) {
            stringBuilder.append(target.get(i));
        }
        long afterTime = System.currentTimeMillis();
        long timeDifference = (afterTime - beforeTime);
        System.out.printf("string builder를 사용할 때 소요 시간 : %d\n", timeDifference);
        return stringBuilder.toString();
    }
```
```
+ 사용할 때 소요 시간 : 67
effective Java Chapter9  item63  문자열 연결은 느리니 주의하라

string builder를 사용할 때 소요 시간 : 0
effective Java Chapter9  item63  문자열 연결은 느리니 주의하라
```

#### 출처 

이펙티브 자바 3/E
