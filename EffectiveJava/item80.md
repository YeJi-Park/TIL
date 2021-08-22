## Effective Java 3/E

#### Item 80. 스레드보다는 실행자, 태스크, 스트림을 애용하라

##### 실행자 프레임워크

- java.util.concurrent 패키지로 유연한 태스크 실행 기능을 담고 있음

```java
ExecutorService exec = Executors.newSingleThreadExecutor(); // 실행자 생성
exec.execute(runnable); // 실행할 태스크를 넘김
exec.shutdown(); // 실행자를 종료
```

- 실행자 서비스의 주요 기능
  - get: 특정 태스크가 완료되기를 기다리기
  - invokeAny/invokeAll: 태스크 모음 중 아무거나 1개 혹은 모든 태스크가 완료되기를 기다림
  - awaitTermination: 실행자 서비스가 종료하기를 기다림
  - ExecutorCompletionService: 완료된 태스크들의 결과를 차례로 받음
  - ScheduledThreadPoolExecutor: 태스크를 특정 시간에 혹은 주기적으로 실행하게 함

----

###### 스레드풀의 종류

- 작업을 둘 이상의 스레드가 처리하게 싶다면 스레드 풀 실행자 서비스를 생성하면 됨

- Executors 정적 팩터리에서 대부분의 실행자를 생성할 수 있도록 제공

- 평범하지 않은 실행자를 원한다면 **ThreadPoolExecutor** 클래스를 직접 사용할 수 있음

  - 스레드 풀 동작을 결정하는 거의 모든 속성을 설정할 수 있음

- **newCachedThreadPool**: 작은 프로그램이나 가벼운 서버에 적합함

  - 요청받은 태스크들이 큐에 쌓이지 않고 즉시 스레드에 위임돼 실행

  - 가용한 스레드가 없다면 새로 스레드를 생성

    → 무거운 프로덕션 서버에서는 새로운 태스크가 도착하는 족족 스레드를 생성하며 자원이 낭비됨

- **newFixedThreadPool**: 스레드 개수를 고정

-----

###### ㅇㅇ

- **작업 큐를 직접 만들거나, 스레드를 직접 다루는 일은 삼가야 함**

- 스레드를 직접 다루면 Thread가 작업 단위와 수행 메커니즘 역할을 모두 수행하게 됨

- **실행자 프레임워크에서는 작업 단위(태스크)와 실행 메커니즘이 분리됨**

  - 태스크: Runnable, Callable

    > Callable은 Runnable과 비슷하지만 값을 반환하고 임의의 예외를 던질 수 있음

  - 실행 매커니즘: 실행자 서비스

  - 태스크 수행을 실행자 서비스에 맡기면 태스크 수행 정책을 선택할 수 있고, 이후에 변경할 수 있음

    → **실행자 프레임워크가 작업 수행을 담당함**

-----

###### ForkJoinTask

- 자바7부터 실행자 프레임워크에서 지원됨

- ForkJoinPool이라는 실행자 서비스가 실행해줌

- ForkJoinTask는 작은 하위 태스크들로 나뉠 수 있고, ForkJoinPool을 구성하는 스레드들이 이 태스크들을 처리

- 작업을 먼저 끝낸 스레드는 다른 스레드의 남은 태스크를 가져와 처리함

- ForkJoinTask를 작성하고 튜닝하기는 어렵지만 ForkJoinPool을 이용해 만든 병렬 스트림을 이용하면 적은 노력으로 이점을 얻을 수 있음

  > fork-join에 적합한 형태의 작업이어야 함

