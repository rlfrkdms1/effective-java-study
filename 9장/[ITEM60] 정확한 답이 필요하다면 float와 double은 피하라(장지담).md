# float과 double의 단점

- float과 double은 과학, 공학 계산용으로 근사치를 계산한다
- 정확한 계산, 금융 계산에는 사용하지 말자
- 10의 음의 거듭제곱 형태를 표현할 수 없다.

```java
    public static void main(String[] args) {
        double funds = 1.00;
        int itemsBought = 0;
        for (double price = 0.10; funds >= price; price += 0.10) {
            funds -= price;
            itemsBought++;
        }
        System.out.println(itemsBought + "개 구입");
        System.out.println("잔돈(달러): " + funds);
    }
```
![image](https://github.com/rlfrkdms1/effective-java-study/assets/96513365/08aea7a2-d606-40b9-891a-d52cd33ff5a1)
부정확한 결과를 얻었다. 

# BigDecimal, int, long을 사용하자
금융 계산에는 BigDecimal, int, long을 사용하자.

```java
    public static void main(String[] args) {
        final BigDecimal TEN_CENTS = new BigDecimal(".10");

        int itemsBought = 0;
        BigDecimal funds = new BigDecimal("1.00");
        for (BigDecimal price = TEN_CENTS;
             funds.compareTo(price) >= 0;
             price = price.add(TEN_CENTS)) {
            funds = funds.subtract(price);
            itemsBought++;
        }
        System.out.println(itemsBought + "개 구입");
        System.out.println("잔돈(달러): " + funds);
    }
```
- BigDecimal을 사용하면 올바른 결과를 얻는다.
- 문자열을 받는 생성자를 활용햐 부정확한 값 사용을 막았다. 
- 기본 타입보다 사용이 불편하다
- 느리다.

```java
    public static void main(String[] args) {
        int itemsBought = 0;
        int funds = 100;
        for (int price = 10; funds >= price; price += 10) {
            funds -= price;
            itemsBought++;
        }
        System.out.println(itemsBought + "개 구입");
        System.out.println("잔돈(센트): " + funds);
    }
```
- 대안으로 기본 타입 int를 사용할 수 있다.

# 무엇을 사용해야 할까?
- 성능이 중요하지 않고, 코딩 시 불편함을 감수할 수 있고, BigDecimal이 제공하는 반올림 모드를 활용하고 싶다면 BigDecimal을 사용하자
- 성능이 중요하고 숫자가 너무 크지 않으면 int, long을 사용하자. 
- 아홉 자리 십진수까지는 int, 열여덟 자리 십진수까지는 long, 초과는 BigDecimal을 사용하자. 
