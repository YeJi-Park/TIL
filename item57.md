## Effective Java 3/E

#### Item 57. 지역변수의 범위를 최소화하라

##### 지역변수의 사용

- 지역변수의 유효 범위를 최소로 줄이면 **코드 가독성과 유지보수성이 높아지고 오류 가능성이 낮아짐**
- 가장 처음 쓰일 때 선언할 것
  - 미리 선언해두면 가독성이 떨어지고, 사용 시점에는 값이 기억이 나지 않을 수 있어 오류 생길 수 있음
  - 사용 후에도 메모리가 살아있을 수도 있으므로 의도한 동작이 아니라 오류를 일으킬 수 있음

- 선언과 동시에 초기화해야 함

  - 초기화에 필요한 정보가 충분하지 않다면 **충분해질 때 까지 선언을 미룰 것**

  - try-catch는 예외: 초기화 중에 예외를 던질 가능성이 있다면 try 블록에서 초기화해야 함

    > 메서드까지 예외가 전파되지 않도록

- 메서드를 작게 유지하고 한 가지 기능에 집중할 것

  > 기능 별로 메서드를 쪼개자

###### 반복문에서의 변수 사용

```java
// 일반적인 반복문
for( Element e : c){
    ... // do something
}

// 반복자의 메소드 사용이 필요할 때
for(Iterator<Element> i = c.iterator; i.hasNext(); ){
    Element e = i.next();
    ... // do something
}
```

- 반복 변수의 범위를 반복분의 몸체, for 키워드와 몸체 사이의 괄호 안으로 제한
- 반복 변수를 반복문 종료 후에도 사용하는 게 아니라면 while보단 for을 사용

```java
Iterator<Element> i = c.iterator; 
while(i.hasNext()){
    doSomething(i.next());
}

Iterator<Element> i2 = c2.iterator; 
while(i.hasNext()){ // i2가 아니라 i를 사용
    doSomething(i2.next());
}
```

- i의 유효범위가 끝나지 않았기 때문에 i2가 아니라 i를 사용해도 오류임을 알 수 없음

- for를 사용하면 반복 변수의 유효범위가 for문과 일치하기 때문에 영향을 주지 않음

  