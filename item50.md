## Effective Java 3/E

#### Item 50. 적시에 방어적 복사본을 만들라

- 네이티브 언어: OS에 의해 직접적으로 컴파일 되는 코드

  > C, C++

- 매니지드 언어: 중간언어를 사용해 OS에 비종속적으로 사용할 수 있는 언어

  > Java, Python 등 인터프리터 언어?

  -----

- 매니지드 언어는 안정적인 것이 장점이지만 언제나 불변식이 유지되는 것은 아님

  → 방어적인 프로그래밍이 필요

###### 다른 클래스의 불변식을 깨지게 만들 수 있는 예

```java
public final class Period {
	private final Date start;
	private final Date end;
	
	public Period(Date start, Date end) {
		if(start.compareTo(end) > 0)
			throw new IllegalArgumentException();
		this.start = start;
		this.end = end;
	}

	public Date getStart() {
		return start;
	}

	public Date getEnd() {
		return end;
	}
}

public class PeriodClient{
	public static void main(String[] args) {
		Period p = new Period(new Date(), new Date());
		p.getStart().setYear(2015); // Deprecate됨
	}
}
```

- start, end는 final로 선언되어 있고 생성자에서 제약을 체크하고 있어 불변으로 생각할 수 있지만,

  Date 자체가 가변이기 때문에 불변식이 깨질 수 있음

  > Instant, LocalDateTime 등의 불변인 클래스를 사용

###### 방어적 복사

```java
	public Period(Date start, Date end) {
        // 복사 수행
		this.start = new Date(start.getTime()); 
		this.end = new Date(end.getTime());
		
        // TOCTOU 공격을 막기 위해 복사 후 비교
		if(this.start.compareTo(this.end) > 0)
			throw new IllegalArgumentException();
	}

	// 접근자를 통해 가변 정보에 접근하는 것을 막기 위해 새로운 복사본을 반환
	public Date getStart() {
		return new Date(start.getTime());
	}

	public Date getEnd() {
		return new Date(end.getTime());
	}
```

- **final 클래스가 아닌 경우 Clone해서 복사하는 것이 아니라, 새로운 인스턴스를 생성해야 함**
- 접근자를 통해 가변 정보에 접근하는 경우도 고려해야 함

---

###### 언제 방어적 복사?

1. 수신한 객체의 참조를 내부의 자료구조에 보관해야 할 경우 객체가 잠재적으로 변경될 여지가 있는지 검토
2. 변경될 여지가 있다면, 변경되어도 문제 없는지 확인
3. 확신할 수 없다면 방어적 복사가 필요함

- 내부 객체를 클라이언트에 반환할 때도 마찬가지로 방어적 복사본을 고려해야 함

- 매개변수로 넘김으로써 통제권을 이전하는 경우 방어적으로 복사해 저장할 필요는 없음

  > 이 경우에는 Caller가 더 이상 객체를 수정하지 않는다는 것을 보장해야 함

- **상호 신뢰할 수 있거나 불변식이 깨지더라도 클라이언트에게만 영향이 갈 때를 제외하고는 방어적 복사를 사용해야 함**

  > ex. 래퍼 클래스 패턴

