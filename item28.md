## Effective Java 3/E

#### Item 28. 배열보다는 리스트를 사용하라

##### 배열과 제네릭 타입의 차이

| #    | 배열              | 리스트                                                       |
| ---- | :---------------- | ------------------------------------------------------------ |
| 1    | 공변: 함께 변한다 | 불공변                                                       |
| 2    | 실체화됨          | 런타임에 타입 정보가 소거<br />(제네릭 도입 전과의 호환을 위해) |

> 1) 배열: sub가 super의 하위 타입이라면 sub[]는 super[]의 하위 타입
>    리스트: list<sub>와 list<super>는 다른 타입
>
> ```java
> Object[] obArr = new Long[1];
> obArr = "str"; // Long에 String을 넣으면 런타임에 ArrayStoreException 발생
> 
> List<Object> obList = new ArrayList<>();
> obList.add("str"); // 컴파일 타임에 오류 인지 가능
> ```

------

##### 제네릭 배열이 가능하다면?

- 배열은 제네릭 타입, 매개변수화 타입, 타입 매개변수로 사용할 수 없음

  > new List<E>[], new List<String>[], new E[] 같은 사용 X

  → **타입 안전하지 않기 때문** 

```java
List<String>[]	strLists = new List<String>[];	// (1)
//	List.of(): Returns an immutable list containing an arbitrary number of elements.
List<Integer> intList = List.of(42); 		   // (2)
Object[] objects = strLists;				  // (3)
objects[0] = intList;						 // (4)
String s = strLists[0].get(0);				  // (5)
```

1. 제네릭 배열 생성
2. 원소가 Integer 1개인 리스트 생성
3. Object 배열에 List 배열 할당: List는 Object의 서브타입이므로 가능
4. List<Integer> 를 Object 배열에 저장: List는 런타임에 타입이 소거되므로 가능
5. ClassCastException 발생

------

##### 실체화 불가 타입

- 실체화되지 않아 런타임에 컴파일타임보다 더 적은 타입 정보를 갖는 타입

  > E, List<E>, List<String> 등 

- 매개변수화 타입 중에는 비한정적 와일드카드만 실체화 가능



------

##### 결론

- 배열로 형변환할 때, 제네릭 배열 생성 오류나 비검사 형변환 경고가 뜨는 경우 annotation을 사용해 삭제 가능
- 하지만 성능 면에서 약간 느릴지라도 안정적인 List를 사용하자