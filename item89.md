## Effective Java 3/E

#### Item 89. 인스턴스 수를 통제해야 한다면 readResolve보다는 열거 타입을 사용하라

- 싱글톤 패턴으로 만들어진 클래스는 Serializable을 추가하는 순간 더 이상 싱글톤이 아니게 됨
- 기본 직렬화를 쓰지 않더라도, 명시적인 readObject를 제공하더라도 이 클래스가 초기화될 때 만들어진 인스턴스와는 별개인 인스턴스를 반환하게 됨

###### readResolve

- readObject가 만들어낸 인스턴스를 다른 것으로 대체할 수 있음

- 역직렬화한 객체의 클래스가 readResolve 메서드를 적절히 정의해뒀다면 역직렬화 후 새로 생성된 객체를 인수로 이 메서드가 호출되고, **이 메서드가 반환한 객체 참조가 새로 생성된 객체를 대신해 반환**

  → 대부분의 경우 새로 생성된 객체의 참조는 유지되지 않으므로 가비지 컬렉션 대상이 됨

```java
private Object readResolve() {
	return INSTANCE;
}
```

- 새롭게 역직렬화되어 생성된 객체는 무시하고 기존의 싱글톤 INSTANCE 객체를 반환

- Elvis 인스턴스의 직렬화 형태는 아무런 실 데이터를 가질 이유가 없으니 모든 인스턴스 필드를 transient로 선언해야 함

- **readResolve 메서드를 인스턴스 통제 목적으로 사용한다면 객체 참조 타입 인스턴스 필드는 모두 transient로 선언해야 함**

  → readReslove 메서드가 수행되기 전에 역직렬화된 객체의 참조를 공격할 여지가 남음

  싱글톤이 transient가 아닌 참조 필드를 가지고 있다면 그 필드의 내용은 readResolve 메서드가 실행되기 전에 역직렬화됨

  → 잘 조작된 스트림을 통해 해당 참조 필드의 내용이 역직렬화되는 시점에 그 역직렬화된 참조를 훔쳐올 수 있음	

###### stealer 클래스

```java
public class ElvisStealer implements Serializable{
	static Elvis impersonator;
	private Elvis payload;
	
	private Object readResolve() {
		impersonator = payload;
		return new String[] {"A Fool Such As I"};
	}
	
	private static final long serialVersionUID = 0;
}
```

1. stealer 클래스의 인스턴스 필드는 도둑이 숨길 직렬화된 싱글턴을 참조

2. 직렬화된 스트림에서 싱글톤의 비휘발성 필드를 이 도둑의 인스턴스로 교체함
   - 싱글톤은 도둑을, 도둑은 싱글톤을 참조하는 순환고리가 만들어짐
3. 싱글톤이 도둑을 포함하므로 싱글턴이 역직렬화될 때 도둑의 readResolve 메서드가 먼저 호출됨
   - 도둑의 인스턴스 필드에 역직렬화 도중인 readResolve 수행되기 전의 싱글톤 참조가 담겨있게 됨

4. 도둑의 readResolve는 인스턴스 필드가 참조한 값을 정적 필드로 복사하여 readResolve가 끝난 후에도 참조할 수 있게 함

5. 숨긴 transient가 아닌 필드의 원래 타입에 맞는 값을 반환

   > 이 과정을 생략하면 직렬화 시스템이 도둑의 참조를 이 필드에 저장하려 할 때 VM이 ClassCastException을 던짐

-----

###### 인스턴스 통제

- 수작업으로 만든 스트림을 이용해 여러 개의 인스턴스를 만들어낼 수 있음

```java
public class ElvisImpersonator {
	private static final byte[] serializedForm = { ... };
	
	public static void main(String[] args) {
		Elvis elvis = deserialize(serializedForm);
		Elvis impersonator = ElvisStealer.impersonator;
		
		elvis.printFavorites();
		impersonator.printFavorites();
	}
}
```

- 위의 코드를 실행하면 서로 다른 2개의 Elvis 인스턴스를 생성할 수 있게 됨

- favoriteSongs 필드를 transient로 선언하여 문제를 고칠 수 있지만 **Elvis를 원소 하나짜리 열거 타입으로 선언해서 수정할 수 있음**

- readResolve 메서드를 사용해 순간적으로 만들어진 역직렬화된 인스턴스에 접근하지 못하게 하는 방법은 깨지기 쉽고 신경을 많이 써야 하는 작업임

- 직렬화 가능한 인스턴스 통제 클래스를 열거 타입을 통해 구현하면 **선언한 상수 외의 다른 객체는 존재하지 않음을 자바가 보장함**

  > 공격자가 AccessibleObject.setAccessible 같은 특권 메서드를 악용하면 방어가 무력화됨

###### 1개의 원소를 갖는 enum으로 선언해 방어

```java
public enum Elvis implements Serializable {
	INSTANCE;
	private String[] favoriteSongs = {"Hound Dog", "Heartbreak Hotel"};
	
	public void printFavorites() {
		System.out.println(Arrays.toString(favoriteSongs));
	}
	
}
```

> enum으로 선언하면 왜 ..?

###### 인스턴스 통제를 위해 readResolve 사용

- 직렬화 가능 인스턴스 통제 클래스를 작성해야 하는데 컴파일 타임에는 어떤 인스턴스가 있는지 알 수 없는 상황이라면 열거 타입으로 표현하는 것이 불가능

-----

###### readResolve 메서드의 접근성

- final 클래스
  - private로 사용
- 그 외
  - private: 하위 클래스에서 사용할 수 없음
  - package-private: 같은 패키지에 속한 하위 클래스에서만 사용할 수 있음
  - protected/public: 이를 재정의하지 않은 모든 하위 클래스에서 사용할 수 있음
    - 하위 클래스에서 재정의하지 않았을 때, 하위 클래스를 역직렬화하면 상위 클래스의 인스턴스를 생성하여 ClassCastException을 일으킬 수 있음