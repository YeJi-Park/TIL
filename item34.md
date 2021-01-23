## Effective Java 3/E

#### Item 34.  int  상수 대신 열거 타입을 사용하라

##### 정수 열거 패턴

- JAVA가 열거 타입을 지원하기 전에 정수 상수(static final int)를 여러개 묶어서 선언

- 접두어를 사용해서 이름 충돌을 방지

  > APPLIE_FUJI, APPLE_PIPPIN, .......

- 단점

  - 상수 값이 바뀌면 다시 컴파일해야 함
  - 문자열로 출력하기 어려움(의미가 담겨있지 않은 단순 상수이기 때문에)

-----

##### 열거 타입

```java
public enum Apple {FUJI, PIPPIN, GRANNY_SMITH}
public enum Orange {NAVEL, TEMPLE, BLOOD}
```

- 내부적으로는 클래스이기 때문에 다른 언어의 열거 타입보다 강력함

- **상수 마다 자신의 인스턴스를 만들어 public static final 필드로 공개**

- 외부에서 접근할 수 있는 생성자가 없으므로 1개의 인스턴스만 가지는 것이 보장됨

- 장점

  - 컴파일 타입 안정성 제공

  - 열거 타입마다 이름공간을 다르게 사용함

    > 필드의 이름만 공개되고 상수는 클라이언트에게 각인되지 않기 때문에 상수 추가, 순서 변경 등의 이유로 클라이언트가 재컴파일 할 필요 없음

  - toString 메서드를 사용해 출력하기 적합한 문자열로 만들 수 있음

    > fromString 메서드도 같이 만들어주는 것이 좋음

    ```java
    public class WeightTable{
    	public static void main(String[] args){
            double earthWeight = Double.parseDouble(args[0]);
            double mass = earthWeight / Planet.EARTH.surfaceGravity();
            for(Planet p : Planet.values()) // Enum 타입 Planet
                System.out.printf("%s에서의 무게는 %f이다.%n", p, p.surfaceWeight(mass));
        }
    }
    ```

    메서드, 필드를 추가할 수 있음

  - 인터페이스 구현이 가능

  ------

  ##### 열거타입의 예시: 계산기

  ```java
  public enum Operation{
  	PLUS("+") {public double apply(double x, double y){return x+y;} },
      MINUS("-") {public double apply(double x, double y){return x-y;} },
      TIMES("*") {public double apply(double x, double y){return x*y;} },
      DIVIDE("/") {public double apply(double x, double y){return x/y;} };
      
      // 추상메서드이므로 재정의하지 않으면 컴파일 오류로 알 수 있음
      public abstract double apply(double x, double y); 
      
      private final String symbol;
      
      // 정적 필드가 초기화될 때 맵에 추가됨
      private static final Map<String, Operation> stringToEnum =
  			Stream.of(values()).collect(
  					Collectors.toMap(Object::toString, e -> e));
  	// Collectors.toMap(K, V): 맵에 들어갈 키와 값을 생산하는 함수 인자를 받음
      
      Operation(String symbol) {this.symbol = symbol; }
      
      @Override
      public String toString() { return symbol; }
      
      // Optional로 반환해 해당 연산이 null일수도 있음을 암시
      public static Optional<Operation> fromString(String symbol){
      	return Optional.ofNullable(stringToEnum.get(symbol));
      }
  }
  ```

  - **상수 별 동작**에 대해서 정의할 때, 열거타입이 장점을 가짐

    - Switch 문으로 분기할 경우 새로운 상수를 추가할 때에 별도로 case 문을 추가하지 않으면 런타임 오류 발생

      → **열거타입은 메서드를 가지므로 상수 별로 다르게 동작하는 코드를 구현할 수 있음**
      (상수별 메서드 구현)

  - 상수별 데이터를 사용해 toString, fromString 메서드로 쉽게 입출력 가능

    - 열거타입에는 상수 이름을 입력받아 그 이름에 해당하는 상수를 반환해주는 valueOf(String)이 자동 생성됨

  - 열거 타입의 생성자에서는 정적 필드 중 상수 변수(final)에만 접근 가능
    같은 열거타입의 다른 상수에도 접근 불가능

------

##### 상수별 메서드 구현의 단점

- 열거 타입 상수끼리 코드를 공유하기 어려움

  - 보완 방법

    1. 중복된 코드를 상수마다 넣어줌
    2. 필요한 부분을 헬퍼 메서드로 작성한 후 상수가 자신에 필요한 메서드를 호출

    → 두 방법 다 가독성이 떨어져 오류 발생 가능성이 높음

    3. **새로운 상수를 추가할 때 전략(별도의 열거 타입 등)을 선택하게 함**
       → 전략 열거 타입 패턴 사용

------

##### 열거 타입에서의 Switch 문의 사용

- 기존 열거 타입의 상수별 동작을 혼합해서 사용할 수 있게 함

```java
public static Operation inverse(Operation op){
	switch(op){
        case PLUS: return Operation.MINUS;
        case MINUS: return Operation.PLUS;
        case TIMES: return Operation.DIVIDE;
        case DIVIDE: return Operation.TIMES;
            
        default: throw new AssertionError("알 수 없는 연산: "+op);
    }
}
```

------

##### 결론

- **필요한 원소를 컴파일타임에 다 알 수 있는 상수 집합이라면 열거타입을 사용하자!**