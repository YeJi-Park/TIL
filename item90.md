## Effective Java 3/E

#### Item 90. 직렬화된 인스턴스 대신 직렬화 프록시 사용을 검토하라

- Serializable을 구현하는 순간 생성자 이외의 방법으로 인스턴스를 생성할 수 있게 되어 버그, 보안 문제를 일으킬 수 있게 됨

  → **직렬화 프록시 패턴**으로 위험을 줄일 수 있음

-----

##### 직렬화 프록시 패턴

1. 바깥 클래스의 논리적 상태를 정밀하게 표현하는 중첩 클래스를 설계해 private static으로 선언 

   - 이 클래스가 바깥 클래스의 **직렬화 프록시**가 됨

   - 중첩 클래스, 바깥 클래스 모두 Serializable을 구현해야 함

2. 중첩 클래스(직렬화 프록시)를 작성 

###### 중첩 클래스

- 바깥 클래스를 매개변수로 받는 단 한 개의 생성자를 가짐
- 생성자에서 인수로 넘어온 인스턴스의 데이터를 복사함
- 일관성 검사/방어적 복사가 없어도 무방함

```java
public class SerializationProxy implements Serializable {
	private final Date start;
	private final Date end;
	
	private static final long serialVersionID = 1234564891231234156L;
	
	public SerializationProxy(Period p) {
		this.start = p.start;
		this.end = p.end;
	}
}
```

3. 바깥 클래스에 writeReplace를 추가

```java
private Object writeReplace(){
	return new SerializationProxy(this);
}
```

- writeReplace 메서드가 바깥 클래스의 인스턴스 대신 SerializationProxy를 반환하게 함

  → 직렬화가 이뤄지기 전에 바깥 클래스의 인스턴스를 직렬화 프록시로 변환 

- 직렬화 시스템은 바깥 클래스의 직렬화된 인스턴스를 생성해낼 수 없음

4. 바깥 클래스에 readObject 추가

```java
private void readObject(ObjectInputStream s) throws InvalidObjectException{
    throw new InvalidObjectException("프록시가 필요합니다.");
}
```

- 바깥 클래스에 위의 메서드를 추가해 불변식을 훼손하려는 시도를 막을 수 있음

5. 프록시 클래스에 readResolve 메서드 추가

   ```java
   private Object readResolve(){
       return new Period(start, end); // public 생성자 사용
   }
   ```

   - **바깥 클래스와 논리적으로 동일한 인스턴스를 반환**
   - 역직렬화 시에 직렬화 시스템이 직렬화 프록시를 다시 바깥 클래스의 인스턴스로 변환하게 해줌
     - 역직렬화된 인스턴스가 해당 클래스의 불변식을 만족하는지 검사할 또 다른 수단을 강구하지 않아도 됨
     - 정적 팩터리, 생성자가 불변식을 확인해주고 인스턴스 메서드들이 불변식을 잘 지켜준다면 더 추가해야 할 필요 없음

###### 프록시 클래스의 장점

- 직렬화는 생성자를 이용하지 않고도 인스턴스를 생성하는 기능을 제공하는데 이 부분의 위험성을 제거해줌
  - 가짜 바이트 스트림 공격, 내부 필드 탈취 공격을 프록시 수준에서 차단해줌
- 프록시 클래스는 Period의 필드를 final로 선언해도 되므로 불변 클래스를 만들 수도 있음
- 역직렬화한 인스턴스와 원래의 직렬화된 인스턴스의 클래스가 달라도 정상 동작함

###### 프록시 클래스의 한계

- 클라이언트가 멋대로 확장할 수 있는 클래스에는 적용할 수 없음
- 객체 그래프에 순환이 있는 클래스에도 적용할 수 없음
  - 직렬화 프록시만 가졌을 뿐 실제 객체가 만들어진 게 아니라 ClassCastException 발생
- 느림

