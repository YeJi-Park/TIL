## Effective Java 3/E

#### Item 36. 비트 필드 대신 EnumSet을 사용하라

```java
	public static final int STYLE_BOLD				= 1 << 0;
	public static final int STYLE_ITALIC			= 1 << 1;
	public static final int STYLE_UNDERLINE			= 1 << 2;
	public static final int STYLE_STRIKETHROUGH		= 1 << 3;

txt.applyStyles(STYLE_BOLD|STYLE_ITALIC);
```

- 값들이 집합으로 사용될 경우, 정수 열거 패턴으로 나열된 비트를 OR해 집합으로 사용

- 비트연산을 통해 합집합/교집합 등의 집합 연산을 효율적으로 수행할 수 있음

- 문제점

  - 비트 필드를 출력해도 의미를 알기 어려움

  - 최대 몇 비트가 필요한지 API 작성 시에 알고 있어야 함

    → java.util.**EnumSet** 사용!

------

##### EnumSet

- Set 인터페이스를 구현했기 때문에 다른 Set 구현체와 같이 사용할 수 있지만 내부는 **비트벡터**로 구현

  → removeAll, retainAll 과 같은 대량 작업은 비트 연산을 사용하기 때문에 비트 필드와 비슷한 속도

```java
public class Text{
	public enum Style { BOLD, ITALIC, UNDERLINE, STRIKETHROUGH }
    
    public void applyStyles(Set<Style> styles) {}
}
```