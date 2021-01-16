## Effective Java 3/E

#### Item 26. 로 타입은 사용하지 말라

##### 용어 정리

- **제네릭 타입**(generic type): 선언에 타입 매개변수를 사용하는 클래스/인터페이스. 
- **매개변수화 타입(parameterized type)**: 제네릭 타입에 타입 매개변수로 타입 인자를 전달.
- **로 타입**(raw type): 타입 매개변수를 사용하지 않을 때. 
  - 타입 선언에서 제네릭 타입 정보가 없는 것 처럼 동작
  - 제네릭이 도입되기 전의 코드와의 호환을 위한 목적
- **비한정적 와일드 카드 타입**(unbouded wildcard type): 

> List\<E>: 원소의 타입을 나타내는 타입 매개변수 E를 받는 List 인터페이스
> List\<String>: 원소로 String을 받는 List의 매개변수화 타입
> List: List의 로 타입
>
> List<?>: List의 비한정적 와일드카드 타입

----

##### 로 타입 사용

```java
// stamp 인스턴스만 들어가야 하지만 다른 타입의 인스턴스가 들어가도 오류 발생 X
private final Collection stamps = ...;

// 컴파일러가 "unchecked call" 경고
stamps.add(new Coin(...));

// 
for( Iterator i = stamps.iterator(); i.hasNext(); ){
    Stamp stamp = (Stamp) i.next(); // Coin을 꺼내는 시점에서 ClassCastException 발생
    stamp.cancel();
}
```

- Generic이 도입되기 전 Collection 사용. **현재도 동작하지만 권장하지 않음**
- 런타임에 예외가 일어날 수 있으므로 사용하면 안됨

###### 사용 가능한 경우

1. Class 리터럴

   > List.Class  (O)
   >
   > List\<String>.Class  (X)

2. Instanceof 연산자: 런타임에는 제네릭이 지워지기 때문에 적용되지 않음

----

##### 제네릭 타입 사용

```java
// 컴파일러가 Stamp의 인스턴스만 들어가야한다는 것을 인지
private final Collection<Stamp> stamps = ...;

// Coin의 인스턴스를 넣을 경우 컴파일러가 경고
// stamps.add(new Coin(...));
```

- 컴파일 타임에 타입 체킹 가능 -> 에러 방지
- 타입 변환을 제거할 수 있어 성능 향상

-----

##### Object 타입 매개변수와 로 타입의 차이

```java
// Object를 타입 매개변수로 받는 매개변수화 타입 리스트
List<Object> strings = new ArrayList<>();

// 로 타입 리스트
List raw_strings = new ArrayList<>();
```

- List<Object>는 모든 타입을 허용하는 리스트임을 컴파일러에게 전달하는 것

```java
public static void main(Stinrg[] args){
    // Object를 타입 매개변수로 받는 매개변수화 타입 리스트
    List<Object> strings = new ArrayList<>();
    
    List raw_strings = new ArrayList<>();
    
    unsafeAdd_1(strings, Integer.valueOf(42)); 
    String s = strings.get(0); // ClassCastException 발생
    unsafeAdd_2(strings, Integer.valueOf(42));
}

// 로 타입 리스트를 인자로 받기 때문에 경고 
private static void unsafeAdd_1(List list, Object o){
    list.add(o);
}

 // List<Object>와 다른 타입의 리스트이기 때문에 컴파일 오류
private static void unsafeAdd_2(List<String> list, Object o){
    list.add(o);
}
```

List\<String>은 List\<Object>와 다르기때문에 컴파일 타임에 타입 체킹 가능
→ **타입 안정성을 위해 매개변수화 타입을 사용해야 함**



-----

##### 비한정적 와일드카드 타입의 사용

- 로 타입과 다르게 안전함
- null 외의 값은 추가할 수 없음

> 추가) 비한정적 와일드 카드가 사용되는 부분
>
> ```java
> public void printList(Iterable<?> iter){
>     for(Object  item: iter){
>         System.out.println(item);
>     }
> }
> ```
>
> iterable 내부의 타입이 중요하지 않은 경우의 작업에 추상화시켜 사용가능

ref. [ What is the use and point of unbound wildcards generics in Java?](https://stackoverflow.com/questions/7671072/what-is-the-use-and-point-of-unbound-wildcards-generics-in-java)

