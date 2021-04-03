## Effective Java 3/E

#### Item 78. 공유 중인 가변 데이터는 동기화해 사용하라

##### 동기화

###### Synchronized 키워드

- 해당 메서드나 블록을 한번에 한 스레드씩 수행하도록 보장(동기화)

###### 동기화의 기능

1. 배타적 실행

   - 객체를 하나의 일관된 상태에서 다른 일관된 상태로 변화시킴

     > 변경하는 중의 일관되지 않은 순간의 객체를 다른 스레드가 보지 못함

2. 스레드 간 안정적 통신

   - 한 스레드가 만든 변화를 다른 스레드가 수정이 반영된 상태로 안정적으로 읽어옴
   - 자바 언어 명세는 **한 스레드가 저장한 값이 다른 스레드에게 보이는 지는 보장하지 않음**



- 사용자들은 보통 동기화의 **① 배타적 실행** 기능을 중점적으로 생각함

- 자바 명세 상 long, double 외의 변수를 읽고 쓰는 동작은 원자적임

  → 그렇다고 원자적 데이터를 읽고 쓸 때 동기화하지 않으면 문제가 일어날 수 있음

------

##### 동기화에서 제공하는 안정적 통신의 중요성

###### 원자적으로 읽고 쓸 수 있지만 스레드 간 안정적 통신에 실패하는 예시

```java
public class StopThread {
	private static boolean stopRequested;
	
	public static void main(String[] args) 
			throws InterruptedException {
		Thread backgroundThread = new Thread( ()->{
			int i = 0;
			while(!stopRequested)
				i++;
		} );
		
		backgroundThread.start();
		TimeUnit.SECONDS.sleep(1);
		stopRequested = true; // 종료 플래그 설정
	}
}
```

- 1초 이후에 메인 스레드가 stopRequested를 true로 설정해도 BG 스레드가 끝나지 않음
- 종료 플래그인 stopRequested가 boolean 타입이라 원자적이라 동기화를 제거해도 될 것이라 생각하기 쉽지만, 동기화를 하지 않으면 **메인 스레드가 수정한 값을 BG 스레드가 언제 확인 할 지 알 수 없음**

```java
// 원래 코드
while (!stopRequested)
    i++;

// 최적화된 코드
if(!stopRequested)
    while(true)
        i++;
```

- 동기화를 하지 않으면자바 컴파일러가 위처럼 최적화를 할 수도 있음

  > 하지만 디컴파일러로 코드 확인했을 때는 저렇게 최적화를 하진 않는 것 같음^-T..

###### 수정된 stopThread

```java
public class StopThread {
	private static boolean stopRequested;
	
    // 동기화 쓰기
	private static synchronized void requestStop() {
		stopRequested = true;
	}
	
    // 동기화 읽기
	private static synchronized boolean stopRequested() {
		return stopRequested;
	}
	
	public static void main(String[] args) 
			throws InterruptedException {
		Thread backgroundThread = new Thread( ()->{
			int i = 0;
			while(!stopRequested())
				i++;
		} );
		
		backgroundThread.start();
		TimeUnit.SECONDS.sleep(1);
		requestStop();
	}
}

```

- 쓰기/읽기 모두를 동기화하지 않으면 동작(① 배타적 실행, ② 안정적 통신)을 보장하지 않음

-----

##### volatile 키워드

- **항상 최근에 기록된 값을 읽게됨을 보장함**
- 반복문에서 동기화 비용을 줄이면서 속도가 더 빠른 대안이 될 수 있음

```java
public class StopThread {
	private static volatile boolean stopRequested;
	
	public static void main(String[] args) 
			throws InterruptedException {
		Thread backgroundThread = new Thread( ()->{
			int i = 0;
			while(!stopRequested)
				i++;
		} );
		
		backgroundThread.start();
		TimeUnit.SECONDS.sleep(1);
		stopRequested = true; // 종료 플래그 설정
	}
}
```



###### 동기화 없이 volatile 을 사용할 때 주의해야 함

```java
public class TestVolatile {
	private static volatile int nextSerialNumber = 0;
	
	public static int generateSerialNumber() {
		return nextSerialNumber++;
	}
	
	public static void main(String[] args) throws InterruptedException {
		Runnable t = new Runnable() {
			
			@Override
			public void run() {
				for(int i=0; i<3000; i++)
					generateSerialNumber();
			}
		};
		
		Thread t1 = new Thread(t);
		Thread t2 = new Thread(t);
		t1.start(); t2.start();
		
		t1.join(); t2.join();
		
		System.out.println(nextSerialNumber);
	}
}
```

- 증가 연산자(++)가 코드 상으로는 한 줄이지만 실제로는 ① 값 읽기 ② 값 쓰기의 2가지 동작을 수행함
- 동기화를 하지 않을 경우 한 스레드가 ①을 수행하고 ②를 수행하는 사이에 다른 스레드가 읽을 경우 잘못된 결과가 일어남(안전 실패Safety Failure)
- **volatile 키워드를 제거하고 synchronized 키워드를 붙여서 해결할 수 있음**

###### java.util.concurrent.atomic을 사용한 동기화

```java
public class TestVolatile {
	private static AtomicInteger nextSerialNumber = new AtomicInteger(0);
	
	public static int generateSerialNumber() {
		return nextSerialNumber.getAndIncrement();
	}
	
	public static void main(String[] args) throws InterruptedException {
		Runnable t = new Runnable() {
			
			@Override
			public void run() {
				for(int i=0; i<3000; i++)
					generateSerialNumber();
			}
		};
		
		Thread t1 = new Thread(t);
		Thread t2 = new Thread(t);
		t1.start(); t2.start();
		
		t1.join(); t2.join();
		
		System.out.println(nextSerialNumber);
	}
}
```

- volatile은 안정적 통신만 지원하지만 Atomic*은 원자성까지 지원함
- 성능도 동기화보다 우수함

------

##### 가변 데이터가 불러오는 문제 해결하기

- **애초에 가변적 데이터를 공유하지 않는 것이 좋음**(가변 데이터는 단일 스레드에서만 사용)
  - 위와 같이 사용하려면 문서에 남겨 유지보수 과정에서도 정책이 계속 지켜지도록 해야 함
- 사용하려는 프레임워크와 라이브러리를 이해해야 함
  - 외부 코드가 인지하지 못한 복병으로 작용할 수 있음
- 한 스레드가 데이터를 수정한 후 다른 스레드에 공유할 때, **공유하는 부분만 동기화해도 됨**
  - 객체를 수정하지 않는 스레드는 동기화 없이 자유롭게 값을 읽어갈 수 있음
  - 이러한 객체를 **사실상 불변**이라고 부르며, 다른 스레드에 객체를 건네는 행위를 **안전 발행**이라 부름
  - 객체를 정적 필드, volatile 필드, final 필드, 락을 통해 접근하게 하기, 동시성 컬렉션에 저장하는 등의 방식으로 안전 발행할 수 있음

------

###### 참고 및 궁금한 점 해결

1. long, double은 왜 원자적이지 않을까?

   - machine word size보다 커서ㅋㅋㅋㅋㅋㅋㅋㅋㅋ

     > [Writing long and double is not atomic in Java?](https://stackoverflow.com/questions/517532/writing-long-and-double-is-not-atomic-in-java)

2. hoisting? 

   - 변수 및 함수의 선언부를 코드 상단으로 끌어울리는 것
   - 최초 선언 시 초기화했을 경우, 선언과 할당으로 나누어 hoisting

3. [간단한 웹 Java Decompiler](http://www.javadecompilers.com/)

4. 왜 저런 식으로 hoisting 하지?
   - [Why HotSpot will optimize the following using hoisting?](https://stackoverflow.com/questions/9338180/why-hotspot-will-optimize-the-following-using-hoisting)

```java
while (!stopRequested)
    i++;
```

- 루프 내에서 stopRequested를 바꾸지 않으므로 컴파일러가 변수의 비교문을 hoist해버림

- [Consistency of mutable variables between threads, synchronized, and hoisting](https://stackoverflow.com/questions/14331290/consistency-of-mutable-variables-between-threads-synchronized-and-hoisting)

  - synchronized 키워드에 따른 jvm의 optimizing과 관련된 내용

  ```java
  while (!stopRequested) {
      synchronized (lock) {}
      i++;
  }
  ```

  - synchronized 블록에서 아무것도 하지 않음에도 불구하고 hoisting하지 않음

  - JVM의 구현에서 정의하고 있는 알고리즘을 바탕으로 synchronized 블록이 ordering됨

  - 탈출 분석Escape Analysis: 이 객체가 다른 스레드에서 사용할 수 없는지 판단(지역변수라던지..)

    → 이 결과를 바탕으로 여러 스레드에서 사용될 수 없으면 synchronized가 걸려있어도 락을 걸지 않음

  - Lock-Unlock 과정이 메모리에 영향을 크게 미치므로 VM은 synchronized를 잘 삭제하지 않는다는 것 같음...