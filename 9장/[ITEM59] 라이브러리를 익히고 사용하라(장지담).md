# 표준 라이브러리를 사용하지 않는다면?
```java
    static Random rnd = new Random();

    static int random(int n) {
        return Math.abs(rnd.nextInt()) % n;
    }
```
위와 같이 0부터 n사이의 무작위 정수를 얻는 random 메서드를 직접 구현했다고 하자. 3가지 문제를 갖는다
1. n이 그리 크지 않은 2의 제곱수라면 얼마 지나지 않아 같은 수열이 반복된다.
- linear congruential generator의 성질이다. (참고 : https://stackoverflow.com/questions/27779177/effective-java-item-47-know-and-use-your-libraries-flawed-random-integer-meth)
2. n이 2의 제곱수가 아니라면 몇몇 숫자가 평균적으로 더 자주 반복된다.
nextInt가 0,1,2,3을 반환한다고 가정하자. 
- 2의 제곱수가 아닌 %3을 하면 0,1,2,0을 얻는다. 0이 자주 반복된다.
- 2의 제곱수인 %2를 하면 0,1,0,1을 얻는다. 0과 1을 같은 빈도로 얻는다. 
3. 지정한 범위 바깥의 수가 종종 튀어나올 수 있다.
- INTEGER.MIN_VAL : -2_147_483_648, INTEGER.MAX_VAL : 2_147_483_647
- Math.abs(INTEGER.MIN_VAL) = 2_147_483_648으로 int 범위를 초과해 오버플로에 의해 최솟값인 -2_147_483_648로 돌아간다. 

# 표준 라이브러리를 사용하자 
`Random.nextInt(int)`를 사용하면 위의 문제들을 해결해준다.

표준 라이브러리는
- 전문가, 다른 프로그래머들의 지식을 활용할 수 있다.
- 핵심 로직과 관련 없는 일에 시간을 허비하지 않아도 된다
- 따로 노력하지 않아도 지속적으로 개선된다
- 기능이 점점 많아진다
- 내가 작성한 코드가 많은 사람에게 낯익은 코드가 된다

하지만 많은 프로그래머가 직접 구현 방식을 사용한다
- 라이브러리 기능을 모르기 때문이다
- java.lang, java.util, java.io와 하위 패키지들에는 익숙해지자
- 컬렉션 프레임워크, 스트림 라이브러리, java.util.concurrent에도 익숙해지자

> 바퀴를 다시 발명하지 말자. 라이브러리를 사용하려고 시도해보자. 표준 라이브러리로 불가능하다면 서드파티 라이브러리를 활용해보자. 

