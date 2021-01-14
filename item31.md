## Effective Java 3/E

#### Item 31.  한정적 와일드카드를 사용해 API 유연성을 높이라

- 매개변수화 타입은 불공변

  > List<String>는 List<Object>의 하위타입이 아님
  >
  > 리스코프 치환 원칙 ... ?

  → 한정적 와일드카드를 사용해 유연성을 높일 수 있음

-----

##### Item. 29의 Stack을 사용한 예제

0. pushAll 메소드**(소비자)** 추가

```java
public void pushAll(Iterable<E> src) {
	for (E e : src)
		push(e);
} 
```

- E의 하위타입이 들어올 경우 incompatible types 오류 발생

  > Stack<Number>.pushAll(Integer.valueOf(3)

  -> 매개변수를 E의 iterable이 아니라 **E의 하위타입의 iterable**이 되도록 수정

1. 와일드카드 타입으로 수정

```java
	public void pushAll(Iterable<? extends E> src) {
		for (E e : src)
			push(e);
	}
```

-> ? 근데 이렇게 해도 오류남..ㅠㅠ 

2. popAll 메소드**(생산자)** 추가

```java
public void popAll(Collection<? super E> dest) {
	while(!isEmpty())
		dest.add(pop());
}
```



- **유연성을 극대화하려면 원소의 생산자/소비자 용의 입력 매개변수에 와일드카드 타입을 사용해야 함**

- **PECS: Producer-extends, Consumer-super**

------

##### 명시적 타입 인수

- 컴파일러가 타입을 제대로 추론하지 못할 때 사용

  > Set<Number> numbers = Union.<Number>union(integers, doubles);

- JAVA 8 이전엔 컴파일러의 타입추론 능력이 약해서 사용됨

------

##### 타입 매개변수와 와일드카드

- 공통되는 부분이 많아서 메서드 정의할 때 두 가지 다 사용 가능한 경우가 많음
- 메서드 선언에 타입 매개변수가 한 번만 나오면 와일드카드를 사용

```java
public static <E> void swap(List<E> list, int i, int j); // (1) 비한정적 타입 매개변수
public static void swap(List<?> list, int i, int j);	// (2) 비한정적 와일드카드
```

- (2)는 List<?>이기 때문에 null 외의 값을 넣을 수 없어 오류

  → private 헬퍼 메서드를 사용해 와일드카드 타입의 실제 타입을 알려줄 수 있음

```JAVA
private static <E> void swapHelper(List<E> list, int i, int j);
```



- public API로는 단순한 (2)를 사용하되, 실제로는 오류 발생하지 않는 (1) 사용????