## Effective Java 3/E

#### Item 77. 예외를 무시하지 말라

- 예외를 명시하는 이유는 **그 메서드를 사용할 때 적절한 조치를 취해달라는 의미**임

```java
try{
    ...
}catch(SomeException e){
    // 아무것도 안함
}
```

- catch 블록 안을 비워두면 예외를 무시할 수 있지만, 이런 식으로 사용할 경우 예외가 존재할 이유가 없어짐
- 예외를 무시해야 할 때도 있지만, 그렇게 결정했다면 왜 그렇게 결정했는지 주석으로 남기고 예외 변수도 ignored로 바꾸어야 함

```java
Future<Integer> f = exec.submit(planarMap::chromaticNumber);
int numColors = 4;
try{
    numColors = f.get(1L, TimeUnit.SECONDS);
}catch(TimeOutException|ExecutionException ignored){
    // 기본값을 사용한다. 어쩌구
}
```

- 검사와 비검사 모두 적용되는 내용
- 예측할 수 있는 예외 상황이든 프로그래밍 오류든 빈 catch 블록으로 넘기면 오류를 내재한 채 동작하게 됨
  어느 순간 문제의 원인과 아무 상관없는 곳에서 갑자기 오류가 발생할 수 있으므로 주의해야 함
- **예외를 적절히 처리하면 오류를 피할 수 있음**
- 무시하지 않고 바깥으로 전파되게 놔둬도 디버깅 정보를 남긴 채 프로그램이 신속히 중단되게 할 수 있음