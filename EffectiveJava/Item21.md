## Effective Java 3/E

##### Item 21. 인터페이스는 구현하는 쪽을 생각해 설계해라

- 자바 8 부터 기존 인터페이스에 메서드를 추가할 수 있도록 **디폴트 메서드**가 도입

- 디폴트 메서드를 선언하면, 그 인터페이스를 구현하고 디폴트 메서드를 재정의하지 않은 모든 클래스에서 디폴트 메서드 사용됨
  → 기존 구현체와의 연동을 고려해야 함

- 기존 인터페이스에 추가된 디폴트 메서드는 런타임 오류를 일으킬 수 있음

- 기존 인터페이스에 디폴트 메서드로 새 메서드를 추가하는 일은 꼭 필요한 경우가 아니면 피해야 함

  → 인터페이스를 설계할 때는 여전히 세심한 주의를 기울여야 함

------

org.apache.commons.collections4.collection.**SynchronizedCollection**

- 인자로 들어온 collection 객체를 래핑해서 Thread-safe한 collection 객체로 반환

- 디폴트 메서드 removeIf는 동기화에 대해 알지 못하므로, 메서드 호출에 대한 thread-safe를 보장하지 못함

>  4.4 이후로 재정의함

```java
    /**
     * @since 4.4
     */
    @Override
    public boolean removeIf(final Predicate<? super E> filter) {
        synchronized (lock) {
            return decorated().removeIf(filter);
        }
    }
```

