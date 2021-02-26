## Effective Java 3/E

#### Item 54. null이 아닌 빈 컬렉션이나 배열을 반환하라 

- 컬렉션이 비었을 경우 빈 컬렉션이 아니라 null을 반환하면 클라이언트에서 별도의 처리가 필요함

  - 배열도 동일

- 빈 컬렉션 할당에 드는 비용이 신경쓰일 경우

  ```java
  public List<Cheese> getCheeses(){
      return cheeseInStock.isEmpty()? Collections.emptyList()
          : new ArrayList<>(cheeseInStock);
  }
  
  public Cheese[]  getCheeses(){
      return cheeseInStock.toArray(EMPTY_CHEESE_ARRAY);
  }
  ```

  불변 객체를 선언해 사용할 수 있음

  > 사용하기 전 실제 성능 개선 효과를 확인해야 함