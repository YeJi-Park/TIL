## Effective Java 3/E

#### Item 45. 스트림은 주의해서 사용하라

##### 스트림

- 다량의 데이터 처리 작업을 위해 도입

- 개념

  - 스트림: 데이터 원소의 유한/무한 시퀀스
  - 파이프라인: 원소들로 수행하는 연산 단계를 표현하는 개념

- 소스 스트림 → (중간연산)+ → 종단연산의 흐름

- 종단 연산이 수행될 때 평가되므로, 종단 연산에 쓰이지 않는 데이터 원소는 계산에 사용되지 않음

- 기본적으로 순차적으로 수행됨(parallel 메서드 사용해 병렬 실행도 가능)

  > 병렬 연산이 더 효과적인 경우가 많지 않음

-----
