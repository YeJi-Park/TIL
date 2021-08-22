## Effective Java 3/E

#### Item 29.이왕이면 제네릭 타입으로 만들라

###### Object 배열을 사용하는 Stack을 제네릭으로 바꾸기



0. Object 배열을 사용하는 Stack

```java
public class Stack {
	private Object[] elements;
	private int size = 0;
	private static final int DEFAULT_INITIAL_CAPACITY = 16;
	
	public Stack() {
		elements = new Object[DEFAULT_INITIAL_CAPACITY];
	}
	
	public void push(Object e) {
		ensureCapacity();
		elements[size++] = e;
	}
	
	public Object pop() {
		if(size == 0) {
			throw new EmptyStackException();
		}
		Object result = elements[size--];
        elements[size] = null;
		return result;
	}
	
	public boolean isEmpty() {
		return size == 0;
	}
	
	private void ensureCapacity() {
		if(elements.length == size)
			elements = Arrays.copyOf(elements, 2*size + 1);
	}
} 
```

1. 클래스에 타입 매개변수 추가하기

```java
	public Stack() {
		elements = new E[DEFAULT_INITIAL_CAPACITY]; // Cannot create a generic array of E
	}
```



2. 오류 해결하기

   2.1. 우회해서 해결하기(캐스팅)

   ```java
   	@SuppressWarnings("unchecked")
   	// push로 들어온 element는 언제나 E 타입이기 때문에 타입 안정성 보장
   	public Stack() {
   		elements = (E[]) new Object[DEFAULT_INITIAL_CAPACITY];
   	}
   ```

   - 가독성이 좋음

   - 2.2에 비해 빠름

   - 런타임 타입이 컴파일타입과 달라 힙 오염을 일으킴

     

   2.2. elements를 Object로 변경

   ```java
   private Object[] elements;
   
   	public Object pop() {
   		if(size == 0) {
   			throw new EmptyStackException();
   		}
   		
   		@SuppressWarnings("unchecked")
   		E result = (E) elements[size--];
            elements[size] = null;
   		return result;
   	}
   ```

   - 원소를 읽을 때마다 형번환을 해야해서 느림

   - 힙 오염 발생하지 않음

     

------

##### 배열 대신 리스트?

- 모든 경우에 리스트가 배열을 대체할 수 있는 건 아님



+) **Generic에선 왜 기본 타입이 사용 불가능하지?**

