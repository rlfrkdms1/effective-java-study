# 제네릭 메서드
- 메서드도 제네릭으로 만들 수 있다.
- 매개변수화 타입(`List<String>`)을 받는 정적 유틸리티 메서드는 보통 제네릭이다.
- Collections의 binarySearch같은 알고리즘 메서드는 모두 제네릭이다.
```java
    public static <T> int binarySearch(List<? extends Comparable<? super T>> list, T key) {
        if (list instanceof RandomAccess || list.size()<BINARYSEARCH_THRESHOLD)
            return Collections.indexedBinarySearch(list, key);
        else
            return Collections.iteratorBinarySearch(list, key);
    }
```

제네릭이 아닌 로 타입을 활용하는 메서드를 보자.
```java
    public static Set union(Set s1, Set s2) {
        Set result = new HashSet();
        result.addAll(s2);
        return result;
    }
```
![image](https://github.com/rlfrkdms1/effective-java-study/assets/96513365/16b98408-eea1-4d5f-b07d-0cd22888b045)

- 컴파일은 가능하지만 로 타입을 사용해서 경고가 발생한다.

```java
    public static <E> Set<E> union(Set<E> s1, Set<E> s2) {
        Set<E> result = new HashSet<>(s1);
        result.addAll(s2);
        return result;
    }
```
- 제네릭으로 수정하면 타입 안전하다.
- 메서드의 제한자(`public`) 반환 타입(`Set<E>`) 사이에 타입 매개변수(`<E>`)를 선언한다

---

> 불변 클래스를 여러 타입으로 활용하고 싶으면 제네릭 싱글턴 팩터리를 사용하면 된다(ITEM42)





