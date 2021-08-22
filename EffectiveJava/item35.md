## Effective Java 3/E

#### Item 35. ordinal 메서드 대신 인스턴스 필드를 사용하라

- 열거 타입 상수는 대부분 하나의 정숫값에 대응됨

  → ordinal() 메서드로 해당 상수가 그 열거 타입에서 몇 번째 위치인지를 알 수 있음

```java
public enum Ensemble{
	SOLO, DUET, TRIO, QUARTET;
    public int numberOfMusicians() { return ordinal() + 1; }
}
```

- 문제점

  - 상수 선언 순서가 바뀌면 오동작
  - 이미 사용 중인 정수와 값이 같은 상수는 추가할 수 없음
  - 중간에 빈 값을 사용할 수 없음

- 해결책

  - ordinal 대신 인스턴스 필드에 저장해야 함

  ```java
  public enum Ensemble{
  	SOLO, DUET, TRIO, QUARTET;
      
      private final int numberOfMusicians;
      Ensemble(int size){ this.numberOfMusicians = size; }
      public int numberOfMusicians() { return numberOfMusicians; }
  }	
  ```

  