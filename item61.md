## Effective Java 3/E

#### Item 61. 박싱된 기본 타입보다는 기본 타입을 사용하라

- Java의 데이터 타입은 2가지 종류로 나눌 수 있음
  - 각각의 기본 타입에 대응하는 참조 타입이 하나씩 있음
  - 기본 타입: int, double, boolean
  - 참조 타입: Integer, Double, Booean
- 오토박싱과 오토언박싱이 있어 구분없이 사용할 수 있지만 차이가 있으므로 구분해서 선택해야 함

------

##### 기본타입과 박싱 타입의 차이

|        | 기본타입      | 박싱타입            |
| ------ | ------------- | ------------------- |
| 속성   | 값만 가짐     | 식별성을 가짐       |
| 값     | 유효값만 가짐 | null을 가질 수 있음 |
| 효율성 | 상            | 하                  |



##### 박싱 타입의 잘못된 사용

###### 비교

```java
Comparator<Integer> naturalOrder = (i, j) -> (i < j) ? -1 : ( i == j ? 0 : 1);

naturalOrder.compare(new Integer(42), new Integer(42)); // 0이 아니라 1
```

- 원인

  1. 첫 번째 검사 (i < j)는 제대로 작동함
  2. 두 번째 검사 (i == j)에서 식별성을 검사
     - 서로 다른 인스턴스이기 때문에 비교 결과가 false(1)를 리턴
     - 첫 번째 Integer 값이 두 번째보다 크다고 판단

- 해결

  - 지역 변수를 사용해 박싱된 값을 언박싱 해 기본 타입에 대한 비교를 수행해야 함

  ```java
  Comparator<Integer> naturalOrder = (iBoxed, jBoxed) -> {
      int i = iBoxed; int j = jBoxed;
      (i < j) ? -1 : ( i == j ? 0 : 1);
  };
  
  naturalOrder.compare(new Integer(42), new Integer(42)); // 0이 아니라 1
  ```

##### null

```java
public class unbelivable{
    static Integer i;
    public static void main(String[] args){
        if ( i == 42) // NPE
            System.out.println("error");
    }
}
```

- Integer의 초기값은 null이기 때문에 오토언박싱 과정에서 NPE 발생

###### 오토박싱/언박싱

```java
public static void main(String[] args){
    Long sum = 0L;
    for (long i = 0; i<=Integer.INT_MAX; i++)
        sum += i;
    System.out.println(sum);
}
```

- for문에서 매번 박싱/언박싱이 일어나기 때문에 성능이 느려짐

------

##### 박싱된 기본 타입의 사용

- 매개변수화 타입, 매개변수화 메서드의 타입 매개변수로 사용
- 리플렉션을 통해 메서드 호출할 때 사용