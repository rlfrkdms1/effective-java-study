> 실패 원자적 : 호출된 메서드가 실패하더라도 해당객체는 메서드 호출 전 상태를 유지한다

# 실패 원자적으로 만드는 방법

### 불변 객체로 설계한다
- 불변 객체는 태생적으로 실패 원자적이다.
- 불변 객체의 상태는 생성 시점에 고정되어 절대 변하지 않는다

### 작업 수행에 앞서 매개변수의 유효성을 검사한다
- 객체 상태를 변경하기 전, 잠재적 예외의 가능성을 대부분 걸러낸다.
```java
public Object pop() {
    if (size == 0)
        throw new EmptyStackException();
    Object result = elements[--size];
    elements[size] = null; // 다 쓴 참조 해제
    return result;
}
```
- size값을 확인해 스택이 비었다면 예외를 던진다
  - 객체의 상태는 변함이 없다. 실패 원자적이다. 
- if문이 없더라도 스택이 비었다면 ArrayIndexOutOfBoundsException을 던진다
  - 하지만 size가 음수로 변경된다. 실패 원자적이지 않다.
  - 예외의 추상화 수준이 상황에 어울리지 않는다.

### 실패할 가능성이 있는 모든 코드를 객체의 상태를 바꾸는 코드보다 앞에 배치한다 
- 작업 수행에 앞서 매개변수의 유효성을 검사할 수 없을 때 사용한다
```java
    private V put(K key, V value, boolean replaceOld) {
        Entry<K,V> t = root;
        if (t == null) {
            addEntryToEmptyMap(key, value);
            return null;
        }
        int cmp;
        Entry<K,V> parent;
        // split comparator and comparable paths
        Comparator<? super K> cpr = comparator;
        if (cpr != null) {
            do {
                parent = t;
                cmp = cpr.compare(key, t.key);
                if (cmp < 0)
                    t = t.left;
                else if (cmp > 0)
                    t = t.right;
                else {
                    V oldValue = t.value;
                    if (replaceOld || oldValue == null) {
                        t.value = value;
                    }
                    return oldValue;
                }
            } while (t != null);
        } else {
            Objects.requireNonNull(key);
            @SuppressWarnings("unchecked")
            Comparable<? super K> k = (Comparable<? super K>) key;
            do {
                parent = t;
                cmp = k.compareTo(t.key);
                if (cmp < 0)
                    t = t.left;
                else if (cmp > 0)
                    t = t.right;
                else {
                    V oldValue = t.value;
                    if (replaceOld || oldValue == null) {
                        t.value = value;
                    }
                    return oldValue;
                }
            } while (t != null);
        }
        addEntry(key, value, parent, cmp < 0);
        return null;
    }
```
- TreeMap의 원소를 추가하는 put 메서드
- 원소를 추가하려면 TreeMap의 기준에 따라 비교할 수 있어야 한다.
- 객체의 상태를 바꾸는 코드(원소 추가, 예시에서 addEntry)보다 실패할 가능성이 있는 코드(if-else문에서의 비교)를 앞에 배치했다.

### 객체의 임시 복사본에서 작업을 수행한 후 작업이 성공적으로 완료되면 원래 객체와 교체한다
- 데이터를 임시 자료구조에 저장해 작업하는 게 더 빠를 때 적용하기 좋다 (성능상)
- 정렬을 수행하기 전 입력 리스트의 원소들을 배열로 옮겨 담는다
  - 배열이 리스트보다 원소 접근이 빠르다(성능 향상)
  - 정렬에 실패하더라도 입력 리스트는 변하지 않는다(실패 원자성)

### 작업 도중 발생하는 실패를 가로채는 복구 코드를 작성하여 작업 전 상태로 되돌린다
- 주로 디스크 기반의 내구성을 보장해야 하는 자료구조에 쓰인다.

# 항상 실패 원자성을 달성할 수 있는 것은 아니다
- 두 스레드가 동기화 없이 같은 객체를 동시에 수정한다면 객체의 일관성이 깨질 수 있다
  - ConcurrentModificationexception을 잡아냈다고 해서 그 객체가 여전히 쓸 수 있는 상태라고 가정해서는 안된다.
> 스레드 동기화 : 둘 이상의 스레드가 동시에 접근할 수 없게 하는 것
```java
    public static void main(String[] args) throws InterruptedException {
        List<String> list = new ArrayList<>();
        list.add("A");
        list.add("B");
        list.add("C");

        // 첫 번째 스레드 - 리스트 순회하면서 요소 제거
        Thread thread1 = new Thread(() -> {
            try {
                for (String element : list) {
                    System.out.println("Thread 1 - Removing: " + element);
                    list.remove(element);  // ConcurrentModificationException 발생 가능
                }
            } catch (ConcurrentModificationException e) {
                System.out.println(e);
            }

        });

        // 두 번째 스레드 - 리스트 순회하면서 요소 추가
        Thread thread2 = new Thread(() -> {
            for (int i = 0; i < 5; i++) {
                String newElement = "New" + i;
                System.out.println("Thread 2 - Adding: " + newElement);
                list.add(newElement);  // ConcurrentModificationException 발생 가능
            }
        });

        thread1.start();
        thread2.start();

        thread1.join();
        thread2.join();

        System.out.println(list);
    }
```
- 두 스레드가 같은 객체 list를 동시에 수정해 ConcurrentModificationException이 발생했다.
![image](https://github.com/rlfrkdms1/effective-java-study/assets/96513365/706fe50e-8236-46c0-9b4f-35cadf4675e3)
- ConcurrentModificationException을 try-catch로 잡아냈지만, 객체 상태는 변경되었다. 

- Error는 복구할 수 없으므로 AssertionError에 대해서는 실패 원자적으로 만들려는 시도조차 할 필요가 없다
  - Error는 JVM이 자원 부족, 불변식 깨짐 등 더 이상 수행을 계속할 수 없는 상황을 나타낼 때 사용하기 때문이다.
 
# 실패 원자적으로 만들 수 있더라도 항상 그리 해야 하는 것은 아니다
- 실패 원자성을 달성하기 위한 비용이나 복잡도가 아주 큰 연산


> 메서드 명세에 기술한 예외라면 예외가 발생해도 실패 원자성이 지켜져야 한다. 지켜지지 않는다면, 실패 후 객체 상태를 API 설명에 명시해야 한다. 
