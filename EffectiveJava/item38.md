## Effective Java 3/E

#### Item 38.  확장할 수 있는 열거타입이 필요하면 인터페이스를 사용하라

##### 열거타입의 확장

- 기본적으로 열거타입은 확장할 수 없음

  > subType의 원소가 baseType의 원소로 포함되지 않기 때문에 개념 상 열거타입과 어울리지 않음

- 열거타입은 **임의의 인터페이스를 구현**할 수 있기 때문에 이를 사용해 확장 가능한 열거타입 구현 가능

  > 주로 연산 코드에 사용자 확장 연산을 추가할 수 있도록 열어주는 용도로 사용됨

-----

##### Interface를 구현해 확장한 Operation

###### Operation [Interface]

```java
public interface Operation {
	abstract double apply(double x, double y);
}
```

###### BasicOperation [Enum]

```java
public enum BasicOperation implements Operation {
	PLUS("+"){
		public double apply(double x, double y) { return x+y; }
	},MINUS("-"){
		@Override
		public double apply(double x, double y) { return x-y; }
	},TIMES("*"){
		@Override
		public double apply(double x, double y) { return x*y; }
	},DIVIDE("/"){
		@Override
		public double apply(double x, double y) { return x/y; }
	};
	
	......
}
```

###### ExtendedOperation

```java
public enum ExtendedOperation implements Operation {
	EXP("^"){
		@Override
		public double apply(double x, double y) { return Math.pow(x, y); }
	},REMAINDER("%"){
		@Override
		public double apply(double x, double y) { return x%y; }
	};
    
    .......
}
```

- Operation을 열거타입으로 구현함으로써 확장할 수 있음

  → Operation을 사용하도록 작성되어있다면 새로운 확장 연산들도 다 사용 가능

###### 확장된 열거타입의 사용

1. 한정적 타입 토큰 사용

```java
	private static <T extends Enum<T> & Operation> void test(
			Class<T> opEnumType, double x, double y) {
		for (Operation op : opEnumType.getEnumConstants())
			System.out.printf("%f %s %f = %f%n", x, op, y, op.apply(x, y));
	}
```

- Operation과 Enum을 모두 구현한 T에 대해서 연산 수행

2. 한정적 와일드카드 타입 사용

```java
	private static void test2(
			Collection<? extends Operation> opSet, double x, double y) {
		for (Operation op : opSet) {
			System.out.printf("%f %s %f = %f%n", x, op, y, op.apply(x, y));
		}
	}
```

- **1번에 비해 유연**

  > Operation의 collection을 받기 때문에 Operation의 구현체는 다 들어갈 수 있음

- 특정 연산에서는 EnumSet과 EnumMap을 사용할 수 없음

  > #질문1. 왜????

-----

##### 확장 가능한 열거타입의 문제점

- 열거 타입끼리 구현을 상속할 수 없음
  - 아무 상태에도 의존하지 않을 경우: Default 구현으로 인터페이스에 추가
  - 별도의 도우미 클래스, 정적 도우미 메서드 등으로 분리