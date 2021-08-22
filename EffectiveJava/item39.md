## Effective Java 3/E

#### Item 39.  명명 패턴보다 애너테이션을 사용하라

-----

##### 명명패턴

- 어떤 프로그램 요소의 이름을 통해 구분하는 것

- 단점

  1. 오타가 나면 안됨

  2. 올바른 프로그램 요소에 사용되리라 보증할 방법이 없음

     > 메서드 명명패턴일 경우 클래스에 적용되면 의도와 다르게 동작할 여지가 있음

  3. 프로그램 요소를 매개변수로 전달할 방법이 없음

  → **애너테이션**으로 해결가능

-----

##### 애너테이션

###### Test 애너테이션

```java
@Retention(RetentionPolicy.RUNTIME) // 런타임에도 유지되는 애너테이션임
@Target(ElementType.METHOD)			// 메서드 선언에만 사용될 수 있음
public @interface Test {}
```

- 이 애너테이션을 사용한 메서드에 대해 테스트도구가 테스트를 수행
- 메타 애너테이션: 애너테이션을 정의하는 데에 사용되는 애너테이션

###### 테스트할 샘플 클래스

```java
public class Sample {
	@Test public static void m1() {}
	public static void m2() {}
	@Test public static void m3() { throw new RuntimeException("실패"); }
	public static void m4() {}
	@Test public void m5() {}
	public static void m6() {}
	@Test  public static void m7() { throw new RuntimeException("실패");}
	public static void m8() {}
}
```

- Test 애너테이션이 붙은 4개의 메서드에 대해서 테스트도구가 테스트를 수행할 것
  - m3, m7: 실패
  - m1: 성공
  - m5: 애너테이션을 잘못 사용
- 애너테이션 자체가 이 클래스에 영향을 주는 것이 아니라, 이 클래스를 사용하는 프로그램에 추가적인 정보를 제공

###### 테스트 수행

```java
public class RunTests {

	public static void main(String[] args) throws Exception{
		int tests = 0;
		int passed = 0;
		
		Class<?> testClass = Class.forName("item39.Sample"); // throws classNotFoundException
		
		for(Method m: testClass.getDeclaredMethods()) {
			if(m.isAnnotationPresent(Test.class)) { // method에 애너테이션 달려있으면 테스트 수행
				tests++;
				try {
					m.invoke(null);
					passed++;
				}catch (InvocationTargetException e) {
                    // 테스트가 메서드가 던진 예외를 리플렉션이 래핑해서 다시 던짐 
					Throwable exc = e.getCause();
					System.out.println(m + "실패: "+exc);
				}catch (Exception e) {
                    // 잘못된 사용
					System.out.println("잘못 사용된 @Test: "+ m);
				}
			}
			
		}
		System.out.printf("성공: %d, 실패: %d%n", passed, tests-passed);
	}
}

/*
public static void item39.Sample.m3()실패: java.lang.RuntimeException: 실패
잘못 사용된 @Test: public void item39.Sample.m5()
public static void item39.Sample.m7()실패: java.lang.RuntimeException: 실패
성공: 1, 실패: 3
*/
```

- 클래스 이름의 FQN을 받아 Test 애너테이션이 달려있는 메서드에 대해 테스트 수행

###### 특정한 매개변수를 받는 애너테이션의 사용

```java
public @interface ExceptionTest {
	Class<? extends Throwable> value();
}
```

- Class[]의 형태로 배열을 매개변수로 받는 것도 가능

###### 같은 애너테이션을 여러번 사용

- @Repeatable 메타 애너테이션을 달아 같은 애너테이션을 여러 번 사용할 수 있음

  > 컨테이너 애너테이션을 구현해 내부 애너테이션의 배열을 반환하도록 해야 함

- 반복 가능 애너테이션을 여러 개 달면 **컨테이너** 애너테이션 타입이 적용됨

  > 한 개 달면 원래의 애너테이션 타입이 적용됨

  → **처리할 때에 컨테이너 타입/ 기본 애너테이션 타입 두 개를 모두 검사하도록 해야함**

  

