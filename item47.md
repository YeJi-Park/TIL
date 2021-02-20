## Effective Java 3/E

#### Item 47. 반환 타입으로는 스트림보다 컬렉션이 낫다

- 자바 7까지는 원소 시퀀스를 반환하기 위해서 컬렉션 인터페이스나 배열을 사용

- 자바 8부터는 스트림이 등장
  → 스트림은 반복을 지원하지 않으므로 for-each를 사용하고 싶은 사용자는 불편할 수 있음

  > Iterable을 확장하고 있지 않기 때문

-----

##### Stream.iterator에 메서드 참조를 건넨다면?

```java
// 형변환 필요
for(ProcessHandle ph: (Iterable<ProcessHandle>) ProcessHandle.allProcesses()::iterator) {
	doSomething();
}
```

> 질문1. Stream에는 Iterable의 메서드를 다 구현하고 있기 때문에 Iterable\<ProcessHandle>로 형변환이 되는건지?

- 형변환해서 사용은 가능하지만 가독성이 떨어짐

-----

##### 어댑터를 사용해서 Iterable과 Stream을 상호 변환

```java
public static <E> Iterable<E> iterableOf(Stream<E> stream){
	 return stream::iterator;
}
		
public static <E> Stream<E> streamOf(Iterable<E> stream){
	 return StreamSupport.stream(iterable.spliterator(), false);
}
```

- 공개 API 작성에는 스트림, 반복문 사용 모두를 고려해야 함

-----

##### Collection을 반환하자

- Collection은 반복, 스트림을 모두 지원하므로 일반적인 상황에선 Collection이나 하위타입을 반환하는 것이 좋음

- 모든 상황에 표준 컬렉션을 사용할 필요는 없으므로, 표현을 간결하게 할 수 있다면 전용 컬렉션을 구현

  > 인덱싱 같은 경우 모든 부분 집합을 다 컬렉션에 담으면 메모리 낭비이므로 비트 인덱싱을 사용하는 컬렉션을 구현하면 유용

- **컬렉션을 반환하는 게 불가능하면 스트림과 Iterable 중 더 자연스러운 것을 반환**