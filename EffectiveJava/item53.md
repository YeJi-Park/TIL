## Effective Java 3/E

#### Item 53. 가변인수는 신중히 사용하라

##### 가변인수: 1개 이상의 인수가 필요한 경우

###### 잘못 구현된 예

```java
static int min(int... args) {
		if(args.length == 0)
			throw new IllegalArgumentException("1개 이상의 인수가 필요합니다.");
		int min = args[0];
		for(int i=1; i<args.length; i++) min = min<args[i]?min:args[i];
		return min;
	}
```

- 런타임에 실패해야 오류를 알 수 있음

###### 올바른 예

```java
static int min(int a, int... args);
```

- 매개변수를 2개로 받아 1개 이상의 가변인수를 갖도록 사용할 수 있음

------

##### 가변인수와 성능

- 가변인수는 생성 시마다 배열을 할당하고 초기화하기 때문에 비용이 높음

  → 성능에 민감한 메서드일 경우 다중정의를 사용해 가변인수 메서드 사용을 줄일 수 있음

```java
public void foo() {}
public void foo(int a1) {}
public void foo(int a1, int a2) {}
public void foo(int a1, int a2, int a3) {}
public void foo(int a1, int... rest) {}
```

