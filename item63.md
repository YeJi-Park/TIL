## Effective Java 3/E

#### Item 63. 문자열 연결은 느리니 주의하라

- 문자열 연결 연산자(+)로 문자열 n개를 잇는 시간은 n^2에 비례함

  → 불변이기 때문에 두 문자열을 연결할 경우 양쪽의 내용을 모두 복사해야 함

-------

###### String을 사용해 문자열 연결

```java
public String statement(){
    String result = "";
    for(int i=0; i<numItems(); i++)
        result += lineForItem(i); // 문자열 연결
    return result;
}
```

- 매번 복사해서 연결해야 하기 때문에 품목이 많아질 경우 매우 느림
- 성능을 포기하고 싶지 않으면 StringBuilder 사용

###### StringBuilder를 사용해 문자열 연결

```java
public String statement2(){
    StringBuilder b = nnew StringBuilder(numItems()*LINE_WIDTH);
    for(int i=0; i<numItems(); i++)
        b.appennd(lineForItem(i));
    return b.toString();
}
```

- String 연결 연산을 개선했지만, StringBuilder가 여전히 빠름

- String을 사용한 Statement의 수행시간은 품목이 많아질 수록 제곱에 비례
  StringBuilder를 사용한 Statemennt_2는 선형으로 비례해 증가

- Statement_2는 충분한 크기로 초기화해 성능을 최적화함

  > 기본 값을 사용하더라도 StringBuilder가 훨씬 빠름

----

##### 참고 1) StringBuffer와 StringBuilder의 차이

- StrinbBuffer: Thread-Safe. 속도가 느림

- StringBuilder: 동기화를 고려하지 않기 때문에 연산처리가 비교적 빠름\

  

##### 참고 2) 컴파일러의 최적화

- JDK 1.5 이후로는 컴파일러가 String을 StringBuilder로 **자동으로 최적화**해주는 경우가 있음

  - 한 줄에서 String을 더할 경우 합쳐진 문자열로 바꿔줌

  ```java
  String str1 = "A" + "B" + "C"; // str1 = "ABC" 로 컴파일됨
  ```

  - 한 줄에서 상수와 다른 String을 더핮는 것은 StringBuffer의 append, toString으로 변환됨

  ```java
  String str2 = "String" + str1 + "extra data"; 
  // str2 = (new StringBuilder("String").append(str1).append("extra data").toString();
  ```

  - 반복문에서의 더하기도 StringBuilder로 변환되지만 반복문마다 새로운 StringBuilder를 생성하기 때문에 효율이 떨어짐
  - 반복문 안에서 String 연결 연산자를 사용할 경우 다른 최적화 방법을 생각해보자!

  > cr. [jdk1.5에서 String 더하기의 컴파일시의 최적화](https://gist.github.com/benelog/b81b4434fb8f2220cd0e900be1634753)

