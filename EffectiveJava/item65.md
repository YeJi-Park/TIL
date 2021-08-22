## Effective Java 3/E

#### Item 65. 리플렉션보다는 인터페이스를 사용하라 

- 리플렉션(java.lang.reflect)을 사용하면 임의의 클래스에 접근할 수 있음

  - Class 객체가 주어지면 생성자, 메서드, 필드의 인스턴스를 가져올 수 있음

  - 인스턴스로 멤버 이름, 타입, 시그니처 등을 가져올 수 있음

  - 인스턴스 생성, 메서드 호출, 필드 접근 등이 가능

    > Method.invoke는 메서드를 호출할 수 있게 해줌

- 리플렉션을 이용해 컴파일 타임에 존재하지 않는 클래스를 이용할 수 있음

----

##### 리플렉션의 단점

1. 컴파일 타임 타입 검사의 이점을 누릴 수 없음
   - 예외 검사도 불가능
   - 런타임에 오류가 발생할 수 있음
2. 코드 가독성이 떨어짐
3. 성능이 떨어짐
   - 리플렉션을 통한 메서드 호출은 일반 메서드 호출보다 느림

-----

##### 리플렉션의 사용

- 코드 분석 도구, 의존관계 주입 프레임워크처럼 꼭 리플렉션을 사용해야 하는 경우도 있음

- 아주 제한된 형태로만 사용해야 단점을 피하고 이점만 취할 수 있음

  - 컴파일 타임에 이용할 수 없는 프로그램이라면 리플렉션으로 인스턴스를 생성하되, **적절한 상위 클래스/인터페이스로 참조해 사용해야 함**

    

  ```java
  	public static void main(String[] args) {
  		Class<? extends Set<String>> cl = null;
  		try {
  			cl = (Class<? extends Set<String>>)Class.forName(args[0]);			
  		}catch (ClassNotFoundException e) {
  			fatalError("Class not found");
  		}
  		
          // 생성자
  		Constructor<? extends Set<String>> cons = null; 
  		try {
  			cons = cl.getDeclaredConstructor();
  		}catch (NoSuchMethodException e) {
  			fatalError("No constructor without argument");
  		}
  		
          // 인스턴스 생성
  		Set<String> s = null;
  		try {
  			s = cons.newInstance();
  		}catch (IllegalAccessException e) {
  			fatalError("Cannot access constructor");
  		}catch (InstantiationException e) {
  			fatalError("Cannot instantiate class");
  		}catch (InvocationTargetException e) {
  			fatalError("Constructor throw exception"+e.getCause());
  		}catch (ClassCastException e) {
  			fatalError("");
  		}
  		
  		s.addAll(Arrays.asList(args).subList(1, args.length));
  		System.out.println(s);
  	}
  ```

  - 첫번째 인수로 Set의 구현체 클래스를 받아 인스턴스 생성

  - 이후의 인수를 Set에 추가해 출력

    - 출력 순서는 첫 번째 인수에 다라 달라짐

      > HashSet일 경우 무작위
      > TreeSet일 경우 알파벳 순

  - 문제점

    1. 런타임에 6개나 되는 예외를 던짐

       > 리플렉션 없이 생성한다면 컴파일 타임에 잡아낼 수 있음
       > 리플렉션 예외 각각을 잡지 않고 상위 클래스인 ReflectiveOperationException으로 처리할 수 있음

    2. 너무 길어짐

       > 리플렉션 없이는 생성자 호출 1줄이면 됨

-----

###### 결론

- 리플렉션은 런타임에 존재하지 않을 수도 있는 다른 클래스, 메서드, 필드와의 의존성을 관리할 때 적합
- 버전이 다른 외부 패키지를 다룰 때 유용함
  - 최소한의 환경(가장 오래된 버전)을 지원하도록 컴파일한 후 이후 버전의 클래스, 메서드는 리플렉션으로 접근
  - 접근하려는 클래스, 메서드가 런타임에 존재하지 않을 수 있다는 사실을 감안해야 함