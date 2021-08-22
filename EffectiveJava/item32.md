## Effective Java 3/E

#### Item 32.  제네릭과 가변인수를 함께 쓸 때는 신중하라

##### 가변인수

- 메서드에 넘기는 인수의 개수를 클라이언트가 조절할 수 있음

- 사용 예시

```java
// java.util.List
    @SafeVarargs
    @SuppressWarnings("varargs")
    static <E> List<E> of(E... elements) {
        switch (elements.length) { // implicit null check of elements
            case 0:
                @SuppressWarnings("unchecked")
                var list = (List<E>) ImmutableCollections.ListN.EMPTY_LIST;
                return list;
            case 1:
                return new ImmutableCollections.List12<>(elements[0]);
            case 2:
                return new ImmutableCollections.List12<>(elements[0], elements[1]);
            default:
                return new ImmutableCollections.ListN<>(elements);
        }
    }

// java.util.ImmutableCollections
        @SafeVarargs
        ListN(E... input) {
            // copy and check manually to avoid TOCTOU(Time of check to time of use)
            @SuppressWarnings("unchecked")
            E[] tmp = (E[])new Object[input.length]; // implicit nullcheck of input
            for (int i = 0; i < input.length; i++) {
                tmp[i] = Objects.requireNonNull(input[i]);
            }
            elements = tmp;
        }
```

- 내부적으로는 가변인수 메서드를 호출하면 가변인수를 담기위한 배열이 만들어짐

  > T<E>

  → 이 배열은 클라이언트에 노출됨

- 가변인수에 제네릭이나 매개변수화 타입이 포함되면 힙 오염될 수 있다는 컴파일 경고

  > Type safety: Potential heap pollution via varargs parameter args

------

##### 힙 오염

- 예시

```java
	// stringLists는 실제로 List<E>[2]로 받아짐
	// 런타임에는 타입이 사라지기 때문에 무엇이든 넣을 수 있는 제네릭 타입 리스트가 되는?
	static void dangerous(List<String>... stringLists) { 
		List<Integer> intList = List.of(42);
		Object[] objects = stringLists; 	 // Object[]는 List[]의 supertype이기 때문에 가능
		objects[0] = intList;				// !! 힙 오염 !!
		String s = stringLists[0].get(0); 	// ClassCastExeception 발생
		// -> get하면서 자동 형변환
	}
	
	static void use(String...strings) {
		for(String str:strings) { // strings는 String[]로 받아짐
			System.out.println(str);
		}
	}
	
	public static void main(String[] args) {
		use("A", "B", "C"); 					 // ① 
		dangerous(List.of("ABC"), List.of("DEF")); // ②
	}
```

- ① 일반적인 가변인수 메소드
  String... strings는 String[]로 받아짐
- ② 제네릭 가변인수 메소드
  - stringList의 타입은 List<\E>[]
  - Object[]는 List[]의 supertype이기 때문에 저장 가능
  - intList도 Object의 subtype이기 때문에 저장 가능
    → **힙오염 발생**
  - stringList 내부의 값은 string이어야 하기 때문에 get할 경우 ClassCastException 

- 타입 안정성이 깨지기 때문에 제네릭 가변인수를 사용하는 것은 안전하지 않음

------

##### 제네릭 배열의 선언과 제네릭 가변인수의 사용

- 제네릭 배열을 직접 프로그래머가 선언하는 것은 허용되지 않음

  > List<String>[] strLists; // 불가능

- 제네릭 가변인수를 사용하면 내부적으로는 제네릭 배열이 되지만, 금지시키기엔 실무에서 유용하기 때문에 사용 가능하도록 만듦

-----

##### 안전한 제네릭 가변인수 메서드

- 제네릭 가변인수를 사용하면 위처럼 힙 오염 관련 컴파일 경고가 뜸
- 자바7부터 **@SafeVarargs** 사용해 경고 숨길 수 있음 → **메서드 작성자가 타입 안전함을 보장해야함**

- 안전한 제네릭 가변인수 메서드의 요건

  1. 메서드가 이 제네릭 배열에 아무것도 저장하지 않아야 함(매개변수를 덮어쓰지 않아야 함)
  2. 배열의 참조가 밖으로 노출되지 않아야 함(신뢰할 수 없는 코드가 배열에 접근할 수 없어야 함)

  → **매개변수 배열이 호출자로부터 그 메서드로 순수하게 인수들을 전달하기만 한다면 안전함**



###### 제네릭 배열에 저장하지 않고도 타입 안정성이 보장되지 않는 예시

```java
	static <T> T[] toArray(T... args) {
		return args;
	}
	
	static <T> T[] pickTwo(T a, T b, T c) {
		switch(ThreadLocalRandom.current().nextInt(3)) {
		// toArray에게 넘길 T[] 생성
		// -> T가 무엇이어도 담을 수 있게 Object[]로 생성
		case 0: return toArray(a, b); // toArray는 받은 args를 그대로 반환하기 때문에 Object[] 반환
		case 1: return toArray(a, c);
		case 2: return toArray(b, c);
		}
		throw new AssertionError();
	}
	
	public static void main(String[] args) {
		// 반환값인 Object[]이 String[] 과 다르기 때문에 ClassCastException
		String[] attributes = pickTwo("좋은", "빠른", "저렴한");
	}
```

- pickTwo에서 toArray를 호출하면서 매개변수로 넘길 Object[] 생성

- toArray에서 Object[]를 그대로 반환하므로 pickTwo는 Object[] 반환

- String[]은 Object[]의 subtype이므로 ClassCastException 발생

  

  → 제네릭 가변 매개변수 배열에 다른 메서드가 접근하도록 허용하면 안전하지 않음
  **힙 오염을 호출한 쪽의 콜스택으로까지 전이할 수 있음**

  > *질문1)* 이 예시에서 다른 메서드가 접근했다는 건 제네릭 배열은 return 함으로써 pickTwo가 사용 가능하게 만들었다는 의미인가?

###### 제네릭 배열을 다른 메서드가 접근해도 안전한 경우

1. SafeVarargs로 제대로 annotation 사용된 다른 varargs 메서드에 넘기는 것

2. 이 배열 내용의 일부 함수를 호출만 하는 일반 메서드에 넘기는 것

   > *질문 2)* 이 배열 내용의 일부 함수를 호출만 한다? List<E>일 경우 List 공통의 메서드만 사용한다는 뜻인지?

- 안전하게 사용하는 예시

```java
@SafeVarargs
static <T> List<T> flatten(List<? extends T>... lists){ // lists는 List<E>[]
    List<T> result = new ArrayList<>();
    for(List<? extends T> list: lists) // T를 상속한 타입만 들어갈 수 있는 한정적 와일드카드 타입
        result.addAll(list);
    return result;
}
```

- 정리: 제네릭 가변 매개변수를 사용하며 힙 오염 경고가 뜨는 메서드가 있다면, 위의 요건을 충족하고 있는지(안전한지) 점검하고 @SafeVarargs annotation을 달아줄 것

------

##### 배열을 리스트로

- item 28에서처럼 가변 매개변수(실제로는 배열)를 List로 변경해서 사용할 수도 있음

```java
@SafeVarargs
static <T> List<T> flatten(List<List<? extends T>> lists){ // lists는 List<E> ( E 안에 다시 List<E>가)
    List<T> result = new ArrayList<>();
    for(List<? extends T> list: lists) // T를 상속한 타입만 들어갈 수 있는 한정적 와일드카드 타입
        result.addAll(list);
    return result;
}

//
flatten(List.of(list1, list2, list3, list4));
```

- 정적 팩터리 함수 List.of()를 사용해서 안전하게 가변 갯수 인자를 넘길 수 있음

- 약간 느리지만 컴파일러가 이 메서드의 타입 안정성을 검증할 수 있음 -> 컴파일타임의 안정성 보장

  > 사용하는 팩터리 함수의 SafeVarargs가 검증되었다면!

