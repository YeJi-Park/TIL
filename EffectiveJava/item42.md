## Effective Java 3/E

#### Item 42.  익명 클래스보다는 람다를 사용하라

- 예전에는 함수 타입을 표현할 때, 추상 메서드를 한 개 가지고 있는 인터페이스를 사용했음

  → 이 인터페이스를 구현한 것이 **함수 객체**. 특정 함수를 나타내는 데에 사용됨

------

###### 익명 클래스

```java
Collections.sort(list, new Comparator<Integer>() {
	@Override public int compare(Integer o1, Integer o2) {
		return Integer.compare(o1, o2);
	}
});
```

- 이후로는 **익명 클래스** 사용. 익명 클래스로 전략을 구현해서 사용

  → 코드가 너무 길어 함수형 프로그래밍에 적합하지 않음

-----

###### 함수형 인터페이스

```java
// 람다식을 함수 객체로 사용
Collections.sort(list, (s1, s2) -> Integer.compare(s1, s2));

// 람다에 비교자 생성 메서드를 사용
list.sort(comparingInt(String::length);
```

- 자바8에서 추상 메서드 한 개를 가진 인터페이스(**함수형 인터페이스**)를 람다식으로 인스턴스화 할 수 있게 됨

  함수, 익명 클래스와 개념은 비슷하지만 코드가 간결해 이해하기 편함

- 이 경우에 매개변수, 반환값은 컴파일러가 추론하기 때문에 적지 않음

  → 컴파일러가 추론하지 못하는 경우 프로그래머가 직접 명시해야 함

  > 타입을 명시해야 코드가 더 명확할 때만 제외하고는, 람다의 모든 매개변수 타입은 생략
  > 컴파일러가 경고를 낸다면 명시하는 것이 좋음

- 컴파일러가 타입추론에 필요한 타입 정보는 대부분 제네릭에서 얻어옴 → 제네릭을 정확히 사용해야 함

-----

###### 람다로 구현한 Operation enum

```java
public enum Operation {
	PLUS("+", (x,y) -> x+y),
    MINUS("-", (x,y) -> x-y),
    TIMES("*", (x,y) -> x*y),
    DIVIDE("/", (x,y) -> x/y);
    
    private final String symbol;
    private final DoubleBinaryOperator op;
    
    Operation(String symbol, DoubleBinaryOperator op) {this.symbol = symbol; this.op = op; }
    public double apply(double x, double y) { return op.applyAsDouble(x, y);}
    
    @Override public String toString() { return symbol; }
}
```

- 람다를 이용해 열거타입의 인스턴스 필드를 상수 별로 다르게 지정할 수 있음

------

###### 람다의 활용

- 람다는 이름이 없고 문서화할 수 없기 때문에 **코드 줄 수가 길고 명확하지 않으면 람다를 사용해선 안됨**

- 열거 타입 생성자에 넘겨지는 인수들의 타입도 컴파일 타임에 추론됨

  → 생성자 안의 람다는 **인스턴스 멤버에 접근할 수 없음**

- 자기 자신을 참조할 수 없음

  → 자기 자신을 참조해야 할 경우 익명 클래스를 사용해야 함

- 직렬화 형태가 구현 별로 다를 수 있으므로 직렬화하는 일은 삼가야 함

  > 잘 이해가 안되는데?

