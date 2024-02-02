# 최적화 격언

> (맹목적인 어리석음을 포함해) 그 어떤 핑계보다 효율성이라는 이름 아래 행해진 컴퓨팅 죄악이 더 많다(심지어 효율을 높이지도 못하면서). - 윌리엄 울프

> (전체의 97% 정도인) 자그마한 효율성은 모두 잊자. 섣부른 최적화가 만악의 근원이다. - 도널드 크누스

> 최적화를 할 때는 다음 두 규칙을 따르라. 
첫 번째, 하지 마라
두 번째, (전문가 한정) 아직 하지 마라, 다시 말해, 완전히 명백하고 최적화되지 않은 해법을 찾을 때까지는 하지 마라. - M. A. 잭슨

최적화는 좋은 결과보다는 해로운 결과로 이어지기 쉽고, 섣불리 진행하면 특히 더 그렇다. 

# 최적화
성능 때문에 견고한 구조를 희생하지 말자. 
## 빠른 프로그램 보다는 좋은 프로그램을 작성하라

좋은 프로그램이어도 성능이 나오지 않는다면, 그 아키텍처 자체가 최적화할 수 있는 길을 안내해줄 것이다. 

또한, 캡슐화를 통해 개별 구성요소의 내부를 독립적으로 설계할 수 있다. 

설계단계에서 성능을 반드시 염두에 둬야한다. 아키텍처의 결함이라면 시스템 전체를 다시 작성하지 않고는 해결하지 못할 있다. 

## 성능을 제한하는 설계를 피하라

컴포넌트끼리, 혹은 외부 시스템과의 소통은 완성 후에는 변경하기 어렵거나 불가능할 수 있으며 동시에 시스템 성능을 심각하게 제한할 수 있다. 


## API를 설계할 때 성능에 주는 영향을 고려하라. 
- public 타입을 가변으로 만들지 말자
- 컴포지션으로 해결할 수 있음에도 상속으로 설계하지 말자
- 인터페이스도 있는데 굳이 구현타입을 사용하지 말자

```java
public abstract class Component implements ImageObserver, MenuContainer,
                                           Serializable
{
    public Dimension getSize() {
        return size();
    }
    ...
}
```
Component 클래스의 getSize()는 Dimension, 즉 가변 객체를 반환하도록 되어있다. 따라서 `getSize()`를 호출하는 모든 곳에서 Dimension 인스턴스를 방어적으로 복사하느라 새로 생성해야한다. 

Dimension을 불변으로 만들거나, getSize를 getWidth와 getHeight로 나누는 방법도 있다. 아래와 같이 Dimension은 기본 타입인 width와 height로 이루어져 있으므로 Dimension의 값들을 기본 타입으로 따로 반환하는 방법으로 API를 다르게 설계할 수 있을 것이다. 

```java
public class Dimension extends Dimension2D implements java.io.Serializable {

    /**
     * The width dimension; negative values can be used.
     *
     * @serial
     * @see #getSize
     * @see #setSize
     * @since 1.0
     */
    public int width;

    /**
     * The height dimension; negative values can be used.
     *
     * @serial
     * @see #getSize
     * @see #setSize
     * @since 1.0
     */
    public int height;
```

## 성능을 위해 API를 왜곡하는 건 매우 안 좋은 생각이다. 

왜곡된 API와 이를 지원하는 데 따르는 고통은 영원히 계속될 것이다. 

## 각각의 최적화 시도 전후로 성능을 측정하라

프로그램에서 시간을 잡아먹는 부분을 추측하기 어렵기 때문이다. 

프로파일링 도구는 최적화노력을 어디에 집중해야 할지 찾는데 도움을 준다. 

#### 출처

이펙티브 자바 3/E
