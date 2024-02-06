API 설계 요렁들을 모아놓은 ITEM이다!

# 메서드 이름을 신중히 짓자
- 표준 명명 규칙을 따르자
- 이해할 수 있고, 일관되게 짓자
- 개발자 커뮤니티에서 널리 받아들여지는 이름을 짓자
- java 라이브러리의 API 가이드를 참조하자

# 편의 메서드를 너무 많이 만들지 말자 
- 메서드가 너무 많은 클래스나 인터페이스는 익히기 어렵다
- 아주 자주 쓰일 경우에만 만들자
- 확신이 서지 않으면 만들지 말자

# 매개변수 목록을 짧게 유지하자
- 4개 이하가 좋다
- 같은 타입의 매개변수가 연달아 나오는 경우가 해롭다
  - 실수에 취약하다

과하게 긴 매개변수 목록을 줄이는 방법을 알아보자
**1. 여러 메서드로 쪼갠다**
- 쪼개진 메서드 각각은 원래 매개변수 목록의 부분집합을 받는다
```java
List<E> subList(int fromIndex, int toIndex);

int indexOf(Object o);
```
- List에서는 부분리스트의 원소의 인덱스를 구하는 메서드(부분리스트 시작, 끝, 구하려는 원소)를 subList와 indexOf 메서드로 분리한다.
- 직교성을 높여 메서드 개수가 줄어든다
> 직교성이 높다 : 공통점이 없는 기능들이 잘 분리되어 있다. 부분리스트를 구하는 것과 주어진 원소의 인덱스를 얻는 기능은 공통점이 없다. 공통점이 없는 기능들을 분리해 원자적으로 설계하면 그 조합으로 다른 기능을 할 수 있어 결과적으로 메서드 개수가 줄 수 있다.

**2. 매개변수 여러개를 묶어주는 도우미 클래스를 만들자**
- 일반적으로 정적 멤버 클래스로 둔다
- 매개변수 몇개를 독립된 하나의 개념으로 볼 수 있을 때 사용한다
```java
public class CardGame {
    public static class Card {
        int rank;
        String suit;
    }

    public CardGame(List<Card> deck) {

    }

    public void pickCard(Card card, Player player) {

    }
}
```
- Card의 숫자와 무늬를 Card라는 도우미 클래스로 묶었다. pickCard는 원래 매개변수가 3개여야 하지만 2개로 줄었다.

**3. 빌더 패턴 응용**
- 매개변수가 많고 일부는 생략해도 괜찮을 때
- 모든 매개변수를 하나로 추상화한 객체를 정의한다
- 클라이언트에서 이 객체의 세터를 호출해 필요한 값을 설정하게 한다.
- 각 세터는 매개변수 하나 혹은 연관된 몇 개만 설정하게 한다.
- 클라이언트는 필요한 매개변수를 다 설정한 후 execute를 호출해 매개변수들의 유효성을 검사한다. 설정이 완료된 매개변수 객체를 넘겨 계산을 수행한다.

```java
public class CardGame {

    public void inputPerson(Person person) {
    }
}
```

```java
public class Client {
      cardGame.inputPerson(Person.builder().name("jidam").age(26).execute());
    }
}
```

### 매개변수 타입은 클래스보다 인터페이스가 낫다. 
- HashMap이 아닌 Map을 사용하면 TreeMap, ConcurrentHashMap 등 어떤 Map 구현체도 인수로 건넬 수 있다.
- 클래스를 사용하면 특정 구현체만 사용하도록 제한하게 된다.

### boolean보다는 원소 2개짜리 열거 타입이 낫다.
- 메서드 이름 상 boolean을 받아야 의미가 더 명확할 때는 제외
```java
    public void therapy(boolean isPerson){

    }

    public void therapy(Type type){
        
    }
```

```java
public enum Type {
    Person, Animal;
}
```
- 사람인지 아닌지를 boolean보다 Type 열거형으로 사용하자.
```java
        
        // 명확하지 않다 
        hospital.therapy(true);
        
        // 명확하다
        hospital.therapy(Type.Person);
        hospital.therapy(Type.Animal);
```
- 무슨 일을 하는지 명확하다.

```java
public enum Type {
    Person{
        @Override
        void introduce() {
            System.out.println("저는 사람입니다. 아파서 왔어요.");
        }
    }, Animal{
        @Override
        void introduce() {
            System.out.println("저는 동물입니다. 아파서 왔어요.");
        }
    }, Aliean{
        @Override
        void introduce() {
            System.out.println("저는 외계인입니다. 아파서 왔어요.");
        }
    };
    
    abstract void introduce();
}
```
- 추후에 외계인도 포함하게 된다면 enum에 추가해주면 되므로 확장이 쉽다.
- 열거타입별로 필요한 메서드를 열거타입에 넣을 수 있다.
