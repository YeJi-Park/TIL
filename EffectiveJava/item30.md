## Effective Java 3/E

#### Item 30.이왕이면 제네릭 메서드로 만들라

###### Set.union()을 제네릭 메서드로 만들기

0. 로 타입 사용

```java
public static Set union(Set s1, Set s2){
	Set result = new HashSet(s1); // 로 타입 사용해서 경고
    result.addAll(s2);
    return result;
}
```

- 타입 안전한 메서드를 만들어야 함

1. 제너릭 메소드로 수정

```java
public static Set<E> union(Set<E> s1, Set<E> s2){
	Set<E> result = new HashSet(s1);
    result.addAll(s2);
    return result;
}
```

- s1, s2, result의 타입이 모두 같아야만 함

  → 한정적 와일드카드 타입을 사용해 유연하게 만들 수 있음

------

- 제네릭 싱글톤 팩토리를 통해 불변 객체를 여러 타입으로 활용할 수 있음

  > 요청한 타입 매개변수에 맞게 그 객체의 타입을 바꿔주는 정적 팩토리
  > ex. valueOf() 

```java
private static UnaryOperator<Object> IDENTITY_FN = (t) -> t;

@SuppressWarnings("unchecked")
public static <T> UnaryOperator<T> identityFunction(){
    // annotation 없으면 Type safety 경고
	return (UnaryOperator<T>) IDENTITY_FN;
}

public static void main(String[] args) {
	String[] strings = {"A", "B", "C"};
	UnaryOperator<String> sameString = identityFunction();
	
    for(String s: strings)
		System.out.println(sameString.apply(s));

	Number[] nums = {1, 2.0, 3L};
	UnaryOperator<Number> sameNumber = identityFunction();
	
    for(Number n: nums)
		System.out.println(sameNumber.apply(n));
}
```

------

##### 재귀적 타입 한정(Recursive type bound)

- 자기 자신이 들어간 표현식을 사용해 타입 매개변수의 허용 범위를 한정
- 주로 Comparable 인터페이스와 같이 사용됨

```java
public interface Comparable<T>{
	int compareTo(T o);
}

public final class String
    implements java.io.Serializable, Comparable<String>, CharSequence,
               Constable, ConstantDesc {
	......
}
```

- T: Comparable\<T>를 구현한 타입이 비교할 수 있는 원소의 타입

  > String은 Comparable\<String>을 구현했기 때문에 String 자신과 비교 가능

  