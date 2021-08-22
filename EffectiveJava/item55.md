## Effective Java 3/E

#### Item 55. 옵셔널 반환은 신중히 하라

- 값을 반환할 수 없을 때
  1. 예외 던지기: 진짜 예외적인 상황에서만 사용해야 함. 스택 전체를 캡쳐해야 하므로 비용이 큼
  2. null 반환: 별도의 null 처리 코드가 필요함 → 처리하지 않을 경우 이후 어느 시점에서 예외 발생
  3. Optional\<T> 반환

----

##### Optional

- 보통은 T를 반환해야 하지만 특정 조건에서는 아무것도 반환하지 않아야 할 때 사용할 수 있음

  → 예외 던지기보다 유연하며, null보다 오류 가능성이 적음

```java
 public static <E extends Comparable<E>> Optional<E> max(Collection<E> c){
	 if(c.isEmpty())
		 return Optional.empty();
	 
	 E result = null;
	 for(E e: c) {
		 if(result == null || e.compareTo(e) > 0)
			 result = Objects.requireNonNull(e);
	 }
	 
	 return Optional.of(result);
 }

// 스트림 버전
 public static <E extends Comparable<E>> Optional<E> maxStream(Collection<E> c){
	 return c.stream().max(Comparator.naturalOrder());
 }
```

- **옵셔널을 반환하는 메서드에서는 null을 반환해서는 안됨**

###### 옵셔널의 사용

- 옵셔널은 반환 값이 없을 수도 있음을 사용자에게 명시적으로 알려주기 때문에 클라이언트는 이에 대처해야 함

  1. 기본값을 설정

     ```java
     String lastWordInLexicon = max(words).orElse("단어없음");
     ```

  2. 상황에 맞게 예외 던지기

     ```java
     Toy myToy = max(toys).orElseThrow(TemperTantrumException::new)
     ```

     - 팩터리를 전달해, 예외가 발생할 경우에만 예외를 생성하기 때문에 비용을 줄일 수 있음

  3. 값이 채워져있다고 확신하면 그대로 사용

     ```java
     Element lastNobleGas = max(Elements.NOBLE_GASES).get();
     ```

     - 값이 없을 경우 예외 발생

- 컨테이너 타입은 옵셔널로 감싸지 말고 비어있는 컬렉션으로 반환해야 함

- **결과가 없을 수 있으며, 클라이언트가 이 상황을 처리해야 할 때 사용**

  > Optional은 할당하고 사용해야하므로 성능이 중요한 경우에는 성능 측정 후 사용해야 함

- 박싱 타입을 담은 옵셔널은 무겁기 때문에 int, long, double은 OptionalInt 등의 전용 클래스 사용

  > 없는 박싱 타입도 있음

- 옵셔널을 컬렉션의 키, 값, 원소나 배열의 원소로 사용하지 말아야 함

###### 유용한 메서드들

1. orElseGet(Supplier<? extends T> other)

   - 값이 필요할 때만 supplier를 수행해서 생성
   - 상세: [orElse와 orElseGet의 차이] 부분 참고

2. filter(Predicate<? super T> predicate)

   ```java
    String str = "true";
    String T = Optional.of(str)
   		 			.filter( obj -> "true".equals(str))
   		 			.orElse("Not matched");
    
    String F = Optional.of(str)
   		 			.filter( obj -> "false".equals(str))
   		 			.orElse("Not matched");
    
    System.out.printf("%s, %s%n", T, F); // true, Not matched
   ```

   - predicate 값이 false일 경우 Optional.Empty 반환

3. map

   - mapper 함수를 받아 입력 값을 다른 값으로 변환

4. flatMap

   - map과 비슷하지만 Optional로 반환하기 때문에 입력을 다시 Optional로 사용해야 할 경우 사용

5. IfPresent

   - 값이 존재할 경우에만 실행할 함수를 인자로 받음

6. isPresent

   - 옵셔널에 값이 들어있으면 true, 비어있으면 false

7. stream

   - Optional을 stream으로 변환

     > 값이 있으면 그 값이 담긴 스트림
     >
     > 없으면 빈 스트림을 반환

   

-----

##### 참고) orElse와 orElseGet의 차이

###### orElse(T other)

- 값을 받음
- null이 아니어도 수행

```java
String name = null;
String defaultName = getDefault(); // null이 아니어도 default값 할당이 필요함
String result = Optional.orNullable(name).orElse(defaultName);
```



###### orElseGet(Supplier<? extends T> other)

- Supplier를 받음
- null일 경우에만 수행

```java
String name = null;
Supplier<Striing> sup = () -> getDefault(); // null인 경우에만 supplier가 수행됨
String result = Optional.orNullable(name).orElse(sup);
```

cr. [java, optional의 orElse와 orElseGet의 차이](https://cfdf.tistory.com/34)

