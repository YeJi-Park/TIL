## Effective Java 3/E

#### Item 43. 람다보다는 메서드 참조를 사용하라

- **메서드 참조**를 사용하면 람다보다 더 간결하게 함수 객체를 표현할 수 있음

```java
// 람다
map.merge(key, 1, (count, incr) -> count + incr);

// 메서드 참조
map.merge(key, 1, Integer::sum);
```

- 람다로 표현했을 때 너무 길거나 복잡한 경우 메서드를 선언하고 메서드 참조를 이용해 전달할 수 있음

  > 메서드 이름을 지을 수 있고 문서화도 가능

- 람다가 수행할 함수가 같은 클래스에 있는 경우 람다가 더 짧고 간결할 수도 있음

```java
// 메서드 참조
service.execute(BlahBlahLongNameSameClass::action);

// 람다
service.execute(() ->action());
```

-----

##### 메서드 참조의 유형

| 메서드 참조 유형   | 예시                   | 람다식 표현                                          |
| ------------------ | ---------------------- | ---------------------------------------------------- |
| 정적               | Integer::parseInt      | str->Integer.parseInt(str)                           |
| 한정적(인스턴스)   | Instant.now()::isAfter | Instant then = Instant.now()<br />t->then.isAfter(t) |
| 비한정적(인스턴스) | String::toLowerCase    | str->str.toLowerCase()                               |
| 클래스 생성자      | TreeMap<K,V>::new      | () -> new TreeMap<K,V>()                             |
| 배열 생성자        | int[]::new             | len -> new int[len]                                  |

1. 정적: 정적 메서드를 가리키는 메서드 참조

2. 한정적: 참조 대상을 특정하는 인스턴스 메서드 참조

   함수 객체가 받는 인수와 참조되는 메서드가 받는 인수가 같음

3. 비한정적: 참조 대상이 특정되어 있지 않음. 함수객체를 적용하는 시점에 대상 인스턴스가 결정

4. 클래스 생성자

5. 메서드 참조

- 4,5는 팩터리 객체로 사용됨