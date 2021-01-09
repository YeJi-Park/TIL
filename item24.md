## Effective Java 3/E

#### Item 24. 멤버 클래스는 되도록 static으로 만들라

#### 중첩 클래스

- 다른 클래스 안에 정의된 클래스
- 자신을 감싼 외부 클래스 안에서만 사용되어야 함

---

##### 정적 멤버 클래스

- 외부 클래스의 정적 멤버와 똑같은 접근 규칙을 적용받음
- 주로 외부 클래스와 함께 쓰이는 public 도우미 클래스로 사용됨 (ex. 열거 타입)

##### (비정적) 멤버 클래스

- 외부 클래스의 인스턴스와 암묵적으로 연결됨

  > Aggregation?

- 내부 클래스의 인스턴스 메서드에서 _OuterClass.this_와 같은 방식으로 외부 클래스 인스턴스/인스턴스 메소드 접근 가능
  → __외부 클래스의 인스턴스와 독립적이라면 정적 멤버 클래스로 만들어야 함__
  → 숨은 외부 참조가 생겨 GC가 외부 클래스를 수거하지 못할 수 있음

- 어댑터 패턴에서 자주 사용됨

```java
// java.util.HashMap의 Iterator 클래스
    abstract class HashIterator {
        Node<K,V> next;        // next entry to return
        Node<K,V> current;     // current entry
        int expectedModCount;  // for fast-fail
        int index;             // current slot

        HashIterator() {
            expectedModCount = modCount;
            Node<K,V>[] t = table;
            current = next = null;
            index = 0;
            if (t != null && size > 0) { // advance to first entry
                do {} while (index < t.length && (next = t[index++]) == null);
            }
        }

        public final boolean hasNext() {
            return next != null;
        }

        final Node<K,V> nextNode() {
            Node<K,V>[] t;
            Node<K,V> e = next;
            if (modCount != expectedModCount)
                throw new ConcurrentModificationException();
            if (e == null)
                throw new NoSuchElementException();
            if ((next = (current = e).next) == null && (t = table) != null) {
                do {} while (index < t.length && (next = t[index++]) == null);
            }
            return e;
        }

        public final void remove() {
            Node<K,V> p = current;
            if (p == null)
                throw new IllegalStateException();
            if (modCount != expectedModCount)
                throw new ConcurrentModificationException();
            current = null;
            removeNode(p.hash, p.key, null, false, false);
            expectedModCount = modCount;
        }
    }

    final class KeyIterator extends HashIterator
        implements Iterator<K> {
        public final K next() { return nextNode().key; }
    }

// 생략
```



##### 익명 클래스

- 사용되는 시점에 선언되면서 인스턴스가 만들어짐(외부 클래스의 멤버가 아님)
- 쓰임이 제한적임: 정적 팩토리 메서드 구현에 자주 사용됨
- 상수 변수 이외의 정적 멤버는 가질 수 없음

##### 지역 클래스

- 한 메서드 안에서 지역변수처럼 생성되고 사용되는 클래스
- 그래서 언제 사용..?



---

##### 어댑터 패턴

