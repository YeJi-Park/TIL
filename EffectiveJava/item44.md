## Effective Java 3/E

#### Item 44. 표준 함수형 인터페이스를 사용하라

- 람다의 등장으로 템플릿 메서드 패턴보다, **함수 객체를 매개변수로 받는 메서드**가 많이 사용됨

  → 함수형 매개변수 타입을 올바르게 선택해야 함

  표준 함수형 인터페이스(*java.util.function*)를 잘 활용하자

  -----

###### Function 예시(Operation)

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

- *java.util.function.DoubleBinaryOperator*를 사용한 item42 예제

-----

###### LinkedHashMap 예시

- put할 때, removeEldestEntry의 return 값이 true일 경우 가장 오래된 원소를 제거

1. 기본 형태(현재 LinkedHashMap 구현)

```java
protected boolean removeEldestEntry(Map.Entry<K,V> eldest) {
    return size()>100;
}
```

- removeEldestEntry가 인스턴스 메서드이기 때문에 size() 사용 가능

2. 함수형 인터페이스 사용

   ```java
   // Functinal Interface 정의
   @FunctionalInterface
   interface EldestEntryRemovalFunction<K,V>{
       boolean remove(Map<K,V> map, Map.Entry<K,V> eldest);
   }
   
   // 사용
   public class UseCase<K,V> extends LinkedHashMap<K, V>{
   	private EldestEntryRemovalFunction<K, V> removalFunction;
   	
   	public UseCase(EldestEntryRemovalFunction<K, V> removalFunction) {
   		this.removalFunction = removalFunction;
   	}
   	
   	protected boolean removeEldestEntry(Map.Entry<K, V> eldest) {
   		return removalFunction.remove(this, eldest);
   	}
   }
   ```

   - 사용자가 remove를 구현해서 사용
   - 생성자에 전달하는 함수 인터페이스는 인스턴스 메서드가 아니기 때문에 맵 자신도 함수 객체에 전달

3. 표준 함수형 인터페이스 사용

   ```java
   public class UseCase<K,V> extends LinkedHashMap<K, V>{
   	private BiPredicate<Map<K,V>, Map.Entry<K, V>> removalFunction;
   	
   	public UseCase(BiPredicate<Map<K,V>, Map.Entry<K, V>> removalFunction) {
   		this.removalFunction = removalFunction;
   	}
   	
   	protected boolean removeEldestEntry(Map.Entry<K, V> eldest) {
   		return removalFunction.test(this, eldest);
   	}
   }
   ```

-----

###### 주로 사용되는 표준 함수형 인터페이스 종류

| 인터페이스         | 함수 시그니처       | 예                  |
| ------------------ | ------------------- | ------------------- |
| UnaryOperator\<T>  | T apply(T t)        | String::toLowerCase |
| BinaryOperator\<T> | T aaply(T t1, T t2) | BigInteger::add     |
| Predicate\<T>      | boolean test(T t)   | Collection::isEmpty |
| Function<T, R>     | R apply(T t)        | Arrays::asList      |
| Supplier\<T>       | T get()             | Instant::now        |
| Consumer\<T>       | void Accept(T t)    | System.out::println |

- 기본 함수형에 int, long, double로 변형이 생겨서 총 43개의 인터페이스 존재

- 표준 함수형 인터페이스는 대부분 기본 타입만 지원함

  → 박싱된 기본 타입을 넣어 사용하지 말 것

-----

###### 직접 작성한 표준 함수형 인터페이스를 사용해야 하는 경우

- Comparator\<T> 인터페이스 같이,
  	1. 자주 사용되고 이름이 용도를 잘 설명하고 있음
   	2. 반드시 따라야 하는 규약이 있음
   	3. 유용한 디폴트 메서드를 제공함

------

###### @FunctionalInterface 애너테이션의 사용목적

1. 인터페이스가 람다 목적으로 설계된 것임을 알려줌
2. 추상 메서드를 하나만 가지고 있어야 컴파일됨 → 컴파일 타임 안정성 증대
3. 유지보수 과정에서 실수로 메서드를 추가하지 못하도록 할 수 있음

→ 직접 만든 함수형 인터페이스에는 항상 @FunctionalInterface 애너테이션을 사용해야 함

-----

###### 함수형 인터페이스 활용의 주의점

- 서로 다른 함수형 인터페이스를 같은 위치의 인수로 받는 메서들을 오버로딩해선 안됨

```java
@Override
public Future<?> submit(Runnable task) {
	// TODO Auto-generated method stub
	return null;
}

@Override
public <T> Future<T> submit(Callable<T> task) {
	// TODO Auto-generated method stub
	return null;
}
```

