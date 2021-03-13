## Effective Java 3/E

#### Item 62. 다른 타입이 적절하다면 문자열 사용을 피하라

##### 문자열을 사용하기 적합하지 않은 경우

- **문자열을 다른 값 타입을 대체해 사용하지 말아야 함**

  - 입력 데이터로 문자열을 사용하는 경우 데이터가 진짜 문자열일 때에만 사용할 것

  - 수치형이라면 적당한 수치형 타입(int, float, BigInteger ...)으로 변환해 사용

  - '예/아니오'의 답이라면 적절한 열거 타입이나 boolean으로 변환해서 사용

    → **적절한 값 타입이 있다면 그것을 사용하고, 없으면 만들어서 사용**

    

- 열거 타입을 대신해서 사용하지 말 것(item 34)

  

- 혼합 타입을 대신해서 사용하지 말 것

  ```java
  String compoundKey = className + "#" + i.next();
  ```

  - 두 요소를 따로 접근하기 위해서는 파싱해서 사용해야 하므로 느리고 오류 가능성이 있음

  - 구분자(#)가 두 요소 중 한 곳에서 사용되었다면 결과에 오류가 생길 수 있음

    → 전용 클래스를 만들어 사용

- 권한을 표현하기 적합하지 않음

-------

##### String을 잘못 사용 하고 있는 클래스 리팩토링

1. **String으로 권한을 구분**하는 Thread 클래스

```java
public class ThreadLocal{
    private ThreadLocal() {}
    private static void set(String key, Object value); // 현재 스레드의 값을 키로 구분해 저장
    public static Object get(String key); // 키값에 대한 value 반환
}
```

- 스레드에 지역변수를 가질 수 없었던 Java 2 이전에는 스레드에 키를 부여해 변수에 대한 접근 권한을 부여함
- 문자열 키가 전역으로 공유되기 때문에 고유한 키임을 보장하지 못하고 보안 이슈에 취약함

2. **Key 클래스로 권한을 구분**

```java
public class ThreadLocal{
    private ThreadLocal() {}
    
    public static class key{ key(){} } // inner class
    public static Key getKey() { return new Key(); } // 고유한 키 생성
    
    private static void set(String key, Object value);
    public static Object get(String key);
}
```

- 고유한 키를 생성해 주지만 get/set이 정적 메서드일 필요가 없고 톱레벨 클래스인 ThreadlLocal이 필요 없으므로 추가로 리팩토링 가능

3. 리팩토링

   ```java
   public final class ThreadLocal<T>{
       public ThreadLocal();
       public void set(T value);
       public T get();
   }
   ```

   - 매개변수화 타입으로 선언해 타입 안정성을 확보