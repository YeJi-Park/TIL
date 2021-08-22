## Effective Java 3/E

#### Item 79. 과도한 동기화는 피하라

##### 과도한 동기화가 불러오는 문제

- 응답 불가와 안전 실패를 피하려면 동기화 메서드나 동기화 블록 안에서는 제어를 절대로 클라이언트에 양도하면 안됨

  - 동기화된 영역 안에서는 재정의할 수 있는 메서드는 호출해선 안됨

  - 클라이언트가 넘겨준 함수 객체를 호출해서도 안됨

    → 이런 메서드들을 **외계인 메서드**(Alient Method)라고 부름
  
    → 이러한 메서드들이 어떤 일을 하는 지 알 수 없고 통제할 수도 없기 때문에 예외를 일으키거나 교착상태에 빠지거나 데이터가 훼손될 위험이 있음 
  

--------

##### 외계인 메서드를 호출해 동기화 문제(예외)가 생긴 예

###### Observable Set

```java
public class ObservableSet<E> extends ForwardingSet<E> {
	public ObservableSet(Set<E> s){
		super(s);
	}
	
	private final List<SetObserver<E>> observers = new ArrayList<>();
	
	public void addObserver(SetObserver<E> observer) {
		synchronized (observers) {
			observers.add(observer);
		}
	}
	
	public boolean removeObserver(SetObserver<E> observer) {
		synchronized (observers) {
			return observers.remove(observer);
		}
	}
	
	private void notifyElementAdded(E element) {
		synchronized (observers) {
			for(SetObserver<E> observer : observers) {
				observer.added(this, element);
			}
		}
	}
	
	@Override
	public boolean add(E e) {
		boolean added = super.add(e);
		if(added)
			notifyElementAdded(e);
		return added;
	}
	
	@Override
	public boolean addAll(Collection<? extends E> c) {
		boolean added = false;
		for(E element : c)
			added |= add(element);
		return added;
	}
}
```

###### Observer Interface

```java
@FunctionalInterface
public interface SetObserver<E> {
	void added(ForwardingSet<E> set, E obj);
}
```

> 구조적으로 Observer Interface는 BiConsumer와 똑같음
> 이름이 더 직관적이고 추후에 다중 콜백을 지원하도록 확장할 수 있기 때문에 커스텀 함수형 인터페이스를 정의함

- 관찰자들이 addObserver, removeObserver 메서드를 호출해 구독을 신청하거나 해지할 수 있음

###### 정상적으로 동작하는 경우

```java
public static void main(String[] args) {
	ObservableSet<Integer> set = new ObservableSet<>(new HashSet<>());
	
	set.addObserver((s, e) -> System.out.println(e));
	
	for(int i=0; i<100; i++)
		set.add(i);
}
```

- 1부터 99까지 출력

###### 이상 동작하는 예

```java
	set.addObserver(new SetObserver<Integer>() {
		@Override
		public void added(ObservableSet<Integer> set, Integer obj) {
			System.out.println(obj);
			if( obj == 23)
				set.removeObserver(this);
		}
	});
```

> removeObserver 메서드에 함수객체를 넘겨야 하기 때문에 람다가 아니라 익명 클래스 사용

- 0부터 23까지 출력한 후 구독해지하고 종료되어야 할 것 같지만 ConcurrentModificationException이 발생

- 호출 구조

  1. ObservableSet가 notifyElementAdded로 Observers 리스트 순회
  2. Observer의 added 함수 호출
  3. added 함수 내에서 removeObserver 메서드를 호출(obj가 23이므로)
  4. removeObserver에서 Observers 리스트에서 원소를 삭제하려 함

  → 순회 중에 삭제하려 했으므로 허용되지 않은 동작으로 오류 발생함

- 동기화 블록 안에서 동시 수정이 일어나지 않도록 보장하지만, **콜백을 거쳐 되돌아와 수정하는 것까지 막을 수는 없음**

-----

##### 백그라운드 스레드를 사용해 교착상태가 발생하는 예

```java
	set.addObserver(new SetObserver<Integer>() {
		@Override
		public void added(ObservableSet<Integer> set, Integer obj) {
			System.out.println(obj);
			if( obj == 23) {
				ExecutorService exec = 
						Executors.newSingleThreadExecutor();
				try {
					exec.submit(() -> set.removeObserver(this)).get();
				}catch(ExecutionException|InterruptedException ex) {
					throw new AssertionError(ex);
				}finally {
					exec.shutdown();
				}
			}
		}
	});
```

- exec이 removeObserver 메서드를 호출하고 락을 얻으려고 하지만 이미 메인 스레드가 락을 쥐고 있어 락을 획득할 수 없음

- 메인 스레드는 notifyElementAdded 메서드에서 락을 획득한 채로 BG 스레드가 observer를 삭제하기를 기다리고 있음

  **→ 교착상태**

-----

##### 불변식이 깨진 경우

- 위의 2가지 예시에서는 동기화 영역이 보호하는 자원이 일관된 상태였지만 불변식이 임시로 깨질 수도 있음
- 자바의 락은 재진입을 허용하므로 교착상태에 빠지지는 않음

###### 예외 발생

- 외계인 메서드를 호출하는 스레드는 이미 락을 쥐고 있으므로 **락이 보호하는 데이터에 대해 개념적으로 관련이 없는 다른 작업이 진행 중**인데도 다음번 락 획득도 성공함

  → 락이 제대로 동작하고 있지 않음

- 재진입 가능 락은 객체 지향 멀티스레드 프로그램을 쉽게 구현할 수 있게 하지만 **응답 불가(교착 상태) 상황을 안전 실패(데이터 훼손)로 변모시킬 수 있음**

###### 해결

1. 외계인 메서드를 동기화 블록 바깥에서 호출

```java
	private void notifyElementAdded(E element) {
		List<SetObserver<E>> snapShot = null;
		synchronized (observers) {
			snapShot = new ArrayList<>(observers); // 리스트를 복사
		}
		for(SetObserver<E> observer : snapShot) { // 복사한 리스트를 순회
			observer.added(this, element);
		}
	}
```

	- 리스트를 복사해서 사용해 락 없이도 순회할 수 있음
 - 열린 호출Open Call: 동기화 영역 바깥에서 외계인 메서드를 호출하는 것
   	- caller 입장에서 외계인 메서드가 얼마나 오래 실행될 지 알 수 없는데, 동기화 영역 안에서 호출된다면 다른 스레드는 대기해야 함
   	- 실패 방지 효과 외에 동시성 효율을 개선해줌

2. CopyOnWriteArrayList 사용
   - 내부 변경작업은 복사본을 만들어 수행함
   - 수정할 일은 드물고 순회만 빈번히 일어나는 관찰자 리스트 용도로 적합함

- 기본적으로 **동기화 영역에서는 가능한 일을 적게 해야 함**
  - 오래 걸리는 작업이라면 동기화 영역 바깥으로 옮기는 방법을 찾아보아야 함

------

###### 동기화의 성능 측면

- 동기화가 초래하는 진짜 비용은 락을 얻는 데 드는 CPU 시간이 아니라, **경쟁하느라 낭비되는 지연시간**임
- 가상머신의 코드 최적화를 제한한다는 점도 숨은 비용

------

##### 가변 클래스를 작성할 때 지침

- 아래의 두 가지 방법 중 선택할 것

  1. 동기화를 하지 말고, 이 클래스를 동시에 사용해야 하는 클래스가 외부에서 동기화하도록 하기

     > java.util에서 이 방식을 사용 

  2. 동기화를 내부에서 수행

     - 클라이언트가 외부에서 락을 거는 것보다 동시성을 개선할 수 있을 때만 선택할 것

     > java.util.concurrent

     - 내부에서 동기화하기로 했다면 락 분할, 스트라이핑, 비차단 동시성 제어 등의 기법을 사용해 동시성을 높여줄 수 있음

- 여러 스레드가 호출할 가능성이 있는 메서드가 정적 필드를 수정한다면 그 필드를 사용하기 전에 동기해야 함

  > 비결정적 행동도 용인한다면 동기하지 않아도 괜찮음

-----------

###### 참고

1. 재진입 가능 락?

   - java에서는 모든 객체가 고유의 락을 가지고 있음

     → 그래서 synchronized 블럭의 락으로 객체를 넣어줄 수 있는 것

   - 고유 락은 재진입이 가능함(락의 획득이 호출 단위가 아닌 스레드 단위로 일어남)

     → 이미 락을 획득한 스레드는 같은 락을 얻기 위해 대기할 필요 없이 사용할 수 있음

   - [참고](http://happinessoncode.com/2017/10/04/java-intrinsic-lock/)