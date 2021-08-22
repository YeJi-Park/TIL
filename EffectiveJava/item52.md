## Effective Java 3/E

#### Item 52. 다중정의는 신중히 사용하라

- **다중정의한 메서드는 정적**으로 선택되고, **재정의한 메서드는 동적**으로 선택됨

  → 재정의했을 때는 런타임 타입 기준, 다중정의는 컴파일 타임 기준

------

###### 다중정의Overloading

```java
public class CollectionClassifier {
	public static String classify(Set<?> s) {
		return "Set";
	}
	
	public static String classify(List<?> l) {
		return "List";
	}
	
	public static String classify(Collection<?> s) {
		return "Collection";
	}
}
```

- 컴파일 타임 기준으로는 전부 Collection이기 때문에 항상 세 번째 메서드가 불림

###### 올바르게 동작하는 Classifier

```java
public static String classify(Collection<?> c) {
		return c instanceof Set? "집합":
				c instanceof List? "리스트":"그 외";
	}
```

- instanceof는 런타임에 검사하므로 원하는 방식으로 사용 가능

----

##### 혼란을 주지 않고 다중정의 사용하기

- 매개변수가 수 같은 다중정의는 삼갈 것

- 가변인수를 사용할 경우 다중정의 하지 말 것

  → 대신 메서드 이름을 다르게 지어서 사용

  > ObjectOutputStream.writeBoolean, writeInt 등

- 생성자는 정적 팩터리를 사용해 다중 정의를 방지할 수 있음

----

###### 매개변수 수가 같은 다중정의를 구분하는 방법

- 매개변수 중 하나 이상이 근본적으로 달라야 함

  → 두 타입의 값을 형 변환할 수 없음(null 제외)

- 이 조건이 충족되면 런타임 타입으로 호출될 다중정의 메서드가 결정됨

```java
new Thread(System.out::println).start();

ExecutorService exec = Executors.newCachedThreadPool();
// The type of println() from the type PrintStream is void, 
// this is incompatible with the descriptor's return type: Object
exec.submit(System.out::println);
```

- 동일해보이지만 exec의 경우 submit(Callable\<Object>) 이 사용되기 때문에 컴파일 오류

  > Runnable로 명시적 형변환해줄 경우 오류 나지 않음

- println, submit 모두 다중 정의이기 때문에 다중정의 해소 알고리즘이 올바르게 동작하지 않음

- **메서드 다중정의 시에 함수형 인터페이스는 같은 위치에 받아선 안됨**



------

- 암시적 타입 람다식

