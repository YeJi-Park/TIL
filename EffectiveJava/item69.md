## Effective Java 3/E

#### Item 69. 예외는 진짜 예외 상황에만 사용하라

```java
// index의 끝을 exception으로 확인
try{
    int i=0;
    while(true)
        range[i++].climb();
}catch(ArrayIndexOutOfBoundsException e){}

// 일반적인 순회
for( Mountain m : range)
    m.climb();
```

- 배열 접근할 때마다 index가 경계를 넘는지 체크하지 않도록 exception으로 만듦 
  → 실제로는 일반적인 순회가 더 빠름
  1. 예외는 예외 상황에만 사용되므로 최적화에 별로 신경쓰지 않았을 가능성이 큼
  2. 코드가 try-catch 안에 있으면 JVM이 최적화하는 데에 제한이 있음
  3. 배열 순회는 JVM이 중복 검사를 수행하지 않도록 최적화함
  4. 반복문 안에 버그가 있어도 예외가 버그를 숨겨 디버깅을 어렵게 할 수 있음

-----

- 예외는 제어 흐름이 아닌, 오직 예외 상황에만 사용되어야 함

- 잘 설계된 API라면 클라이언트가 정상적인 제어 흐름에서 예외를 사용할 일이 없게 해야 함

  - 특정 상태에서만 호출할 수 있는 상태의존적 메서드를 제공하면 상태 검사 메서드도 같이 제공해야 함

    > iterator의 next(), hasNext()가 각각 이 메서드

    ```java
    // 일반적인 사용
    for(Iterator<Foo> i = collection.iterator(); i.hasNext(); ){
        Foo foo = i.next();
    }
    
    // hasNext() 메서드가 없을 경우
    try{
        Iterator<Foo> i = collection.iterator();
        while(true){
            Foo foo = i.next();
            ...
        }
    }catch(NoSuchElementException e) {}
    ```

    - 클라이언트가 대신 예외를 사용해 확인해야 함

      → 느리고 장황하므로 사용해선 안됨

  - 상태 검사 대신 비어있는 옵셔널 혹은 null 등의 특수한 값을 반환하는 방법을 사용해도 됨

----

###### 예외 대신 제어흐름에서 상황 별 사용할 적절한 방법

- 옵셔널

  - 외부 동기화 없이 여러 스레드가 동시에 접근할 수 있거나 외부 요인으로 상태가 변할 수 있을 때 사용

    >  검사 메서드-상태 의존적 메서드 호출 사이에 객체 상태가 변화할 수 있음

  - 성능이 중요한 상황에서 상태 검사 메서드에서 상태 의존적 메서드의 작업 일부를 중복 수행하는 경우

- 상태 검사 메서드

  - 다른 모든 경우

