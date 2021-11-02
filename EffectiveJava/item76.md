## Effective Java 3/E

#### Item 76. 가능한 한 실패 원자적으로 만들라 

- 호출된 메서드가 실패하더라도 해당 객체는 메서드 호출 전 상태를 유지해야 함(실패 원자적Failure-atomic)

-----

###### 메서드를 실패 원자적으로 만드는 방법

- 불변 객체

  - 메서드가 실패하면 새로운 객체가 안만들어질 뿐 기존 객체가 불안정한 상태에 빠지지 않음

    > 불변 객체는 생성 시점에 고정되어 변하지 않기 때문

- 가변 객체

  1. 작업 수행에 앞서 **매개변수의 유효성을 검사**(item 49)해 내부 상태를 변경하기 전에 잠재적 예외의 가능성을 걸러냄

     ```java
     // Stack.pop
     public Object pop(){
         if(size == 0)
             throw new EmptyStackException();
         Object result = elements[--size];
         elements[size] = null;
         return result;
     }
     ```

     - size의 값을 확인하여 0이면 예외를 던짐
     - size를 체크하는 if문이 없어도 다음 라인에서 size가 음수가 되며 ArrayIndexOutOfBoundsException가 발생하지만, 상황에 적절하지 않은 예외임

  2. 실패할 가능성이 있는 모든 코드를 객체의 상태를 바꾸는 코드보다 앞에 배치하는 방법

     - 계산을 수행해보기 전에 인수의 유효성을 검증할 수 없을 때 1)에 덧붙여 사용
     - TreeMap의 경우
       - 원소들을 어떠한 기준으로 정렬하기 때문에 추가하려면 기준에 따라 비교할 수 있는 타입이어야 함
       - 엉뚱한 타입의 원소를 추가하려 하면 **트리를 변경하기 전에**, 해당 원소가 들어갈 위치를 찾는 과정에서 ClassCastException 발생

  3. 임시 복사본에서 작업을 수행한 다음 성공적으로 완료되면 원래 객체와 교체하는 방식

     - 임시 자료구조에 저장해 작업하는 게 더 빠를 때 적용하기 좋음
     - 정렬 메서드
       - 정렬을 수행하기 전에 입력 리스트의 원소들을 배열로 옮겨 담음
       - 배열이 정렬 알고리즘의 반복문에서 원소들을 접근하기 더 빠르기 때문이지만, 정렬에 실패하더라도 입력 리스트는 변하지 않는 효과를 얻을 수 있음

  4. 작업 중 발생하는 실패를 가로채는 복구 코드를 작성하여 작업 전 상태로 되돌리는 방법

     - 내구도를 보장해야 하는 자료구조에 사용되는데, 자주 사용되는 방법은 아님

------

###### 항상 실패원자성을 달성해야 할까?

- 일반적으로 권장되는 덕목이지만 항상 달성할 수 있는 것은 아님
- 두 스레드가 동기화 없이 같은 객체를 동시에 수정한다면 일관성이 깨질 수 있으므로 Exception을 잡았다고 하더라도 객체를 쓸 수 있는 상태라고 가정해서는 안됨
- Error는 복구할 수 없으므로 AssertionError는 실패 원자적으로 만들 필요도 없음
- 실패 원자성을 달성하기 위한 비용이나 복잡도가 클 경우 항상 달성해야 하는 것은 아님
  - 문제가 무엇인지 알고나면 실패 원자성을 그냥 달성하게 되는 경우가 있음
- **메서드 명세에 기술한 예외라면 예외가 발생하더라도 객체의 상태는 메서드 호출 전과 똑같이 유지해야 함**
  - 이 규칙을 지키지 못하면 실패 시의 객체 상태를 API 명세에 적시해야 함


