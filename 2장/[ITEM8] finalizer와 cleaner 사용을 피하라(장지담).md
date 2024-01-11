# finalizer와 cleaner
java는 객체 소멸자 finalizer와 cleaner를 제공한다. 일반적으로 예측할 수 없고, 불필요하다.
- 수행 시점을 보장하지 않는다
- 수행 여부를 보장하지 않는다
- finalizer 동작 중 발생한 예외는 무시되고, 처리할 작업이 남아도 그 순간 종료된다. (cleaner에서는 이러한 문제가 발생하지 않는다)
- 성능이 떨어진다
- finalizer는 finalizer 공격에 노출되어 보안 문제를 일으킬 수 있다
  - 생성자나 
