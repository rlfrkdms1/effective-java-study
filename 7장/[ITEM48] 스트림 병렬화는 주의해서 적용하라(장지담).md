- 데이터 소스가 stream.iterate거나 중간 연산으로 limit을 쓰면 파이프라인 병렬화로 성능 개선을 기대할 수 없다 .
- 스트림 파이프라인을 마구잡이로 병렬화하면 성능이 나빠질 수 있다

- 스트림의 소스가 ArrayList, HashSet, HashMap, ConcurrentHashMap, 배열, int범위, long범위 일 때 
