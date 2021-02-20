## Effective Java 3/E

#### Item 48. 스트림 병렬화는 주의해서 적용하라

- 동시성 프로그래밍에는 안전성과 응답 가능 상태를 유지하는 것이 중요함

- Stream.iterate나 limit을 사용하면 병렬화로 성능 개선을 기대할 수 없음

  > iterate는 순차적으로 수행되어야 하기 때문. limit도 요소의 순서에 의존하기 때문에

-----

##### 병렬화가 효율적인 경우

- spatial locality가 보장된 경우

  - ArrayList, HashMap, ......, int, long 등

- 종단 연산의 동작 방식이 reduction일 경우

  - reduce, min, max, ...

    > collect 같은 경우는 적합하지 않음

- 조건이 충족되면 바로 반환하는 형식일 경우

  - anyMatch, allMatch, .....

- Sized 스트림인 경우 분할하기 적합하므로 병렬처리가 효과적

  > Filtered 연산이 있는 경우 스트림의 크기를 예측할 수 없어 병렬처리 성능을 예측하기 어려움

-----

##### 효율적인 병렬화 사용

- 직접 구현한 Stream, Iterable, Collection이 효과적으로 병렬화되기 위해서는 spliterator를 재정의하고 성능 테스트를 해보아야 함

  > [spliterator의 소개 및 활용](https://jistol.github.io/java/2019/11/17/spliterator/)

- 잘못 병렬화된 스트림은 성능만이 아닌, 결과 자체가 잘못될 수도 있기 때문에 주의해야 함

- 스트림이 효율적으로 나뉘고, 적절한 종단 연산을 사용하더라도 병렬화에 들어가는 추가 비용을 상쇄할 수 있는 경우에만 사용해야 함

------

##### 참고할 문서들

- [Java8 Parallel Stream, 성능장애를 조심하세요](https://m.blog.naver.com/tmondev/220945933678)

