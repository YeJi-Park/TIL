## Effective Java 3/E

#### Item 73. 추상화 수준에 맞는 예외를 던져라

- 저수준 예외를 처리하지 않고 바깥으로 전파할 경우, 내부 구현 방식을 드러내어 윗 레벨 API를 오염시킬 수 있음
- 다음 릴리즈에서 구현 방식을 바꾸면 다른 예외가 튀어나와 기존 클라이언트 프로그램을 깨지게 할 수 있음

###### 예외 번역

- 상위 계층에서는 저수준 예외를 잡아 자신의 추상화 수준에 맞는 예외로 바꿔 던져야 함

```java
try{
	... // 저수준 추상화
}catch(LowerLevelException e){
    throw new HigherLevelException(...);
}
```

```java
// AbstractSequentialList 
public E get(int index) {
    try {
        return listIterator(index).next();
    } catch (NoSuchElementException exc) {
        throw new IndexOutOfBoundsException("Index: "+index); // 예외 번역!
    }
}
```

- 예외를 번역할 때, 저수준 예외가 디버깅에 도움이 된다면 예외 연쇄를 사용하는 것이 좋음

----

###### 예외 연쇄

- 문제의 근본 원인인 저수준 예외를 고수준 예외에 실어 보내는 방식

  → 별도의 접근자 메서드를 통해 필요하면 저수준 예외를 꺼낼 수 있음

```java
try{
	... // 저수준 추상화
}catch(LowerLevelException cause){
    throw new HigherLevelException(cause);
}
```

- 고수준 예외 생성자는 상위 클래스의 생성자에 cause(저수준 예외)를 건네어 최종적으로 Throwable 생성자까지 건네게 됨

```java
// 예외 연쇄용 생성자
class HigherLevelException extends Exception{
    HigherLevelException(Throwable cause){
        super(cause);
    }
}
```

- 대부분의 표준 예외는 예왜 연쇄용 생성자를 갖추고 있음
- 그렇지 않더라도 thorwable의 initCause 메서드를 이용해 원인을 추가할 수 있음
- 예외 연쇄는 문제의 원인을 프로그램에서 접근할 수 있게 해주며, 원인과 고수준 예외의 스택 추적 정보를 잘 통합해줌

----

###### 예외번역의 사용

- 예외를 무턱대고 전파하는 것보다는 예외 번역이 우수한 방법이지만, 남용해서는 안됨

- 저수준 메서드가 성공하도록 하여 아래 계층에서는 예외가 발생하지 않도록 해야함

  > 상위 계층 메서드의 매개변수 값을 아래 계층 메서드로 건네기 전에 미리 검사하는 등의 방법 사용

- 아래 계층에서 예외를 피할 수 없다면, 상위 계층에서 예외를 처리하여 API 호출자에까지 전파하지 않는 방법도 사용

  > java.util.logging 같은 기능을 활용해 기록해두면 좋음