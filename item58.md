## Effective Java 3/E

#### Item 58. 전통적인 for문보다는 for-each문을 사용하라

###### 전통적인 for 문 사용

```java
for( int i =0; i<a.length; i++){
    ... // do something
}

for(Iterator<Element> i = c.iterator; i.hasNext(); ){
    Element e = i.next();
    ... // do something
}
```

- 반복자와 인덱스 변수는 실제로 필요한 값들이 아니기 때문에 코드 가독성이 떨어짐

- 반복자, 인덱스 등 요소가 늘어나 잘못 사용할 여지가 있음

  → 컴파일러가 오류를 잡아줄 수 없음

- 컬렉션, 배열이냐에 따라 코드 형태가 달라짐

- **for-each 문을 사용할 것**

  > 정식 이름은 향상된 for문

-----

###### for문의 잘못된 사용

```java
enum Suit { CLUB, DIAMOND, HEART, SPADE}
enum RANK { ACE, DEUCE, THREE, FOUR, FIVE, SIX, SEVEN, EIGHT, NINE, TEN, JACK, QUEEN, KING}

static Collection<Suit> suits = Arrays.asList(Suit.values());
static Collection<Rank> ranks = Arrays.asList(Rank.values());

List<Card> deck = new ArrayList<>();
for(Iterator<Suit> i = suits.iterator(); i.hasNext(); )
    for(Iterator<Rank> j = ranks.iterator(); j.hasNext(); )
        deck.add(new Card(i.next(), j.next())); // NoSuchElementException
```

- i의 next는 숫자 하나당 한 번 불려야 하는데 안쪽 반복문에서 불리고 있기 때문에 에러 발생
- 원하는 동작대로 수행하지 않지만 에러가 발생하지 않은 채로 종료될 수도 있음
- for문을 그대로 사용하면서 이 문제를 해결하기 위해서는, 바깥 반복문에 원소를 저장하는 변수를 하나 추가해야 함 

```java
for(Suit suit: suits)
    for(Rank rank: ranks)
        deck.add(new Card(suit, rank));
```

- for each문 중첩하면 간단하게 해결 가능함

-----

##### for each문을 사용할 수 없는 상황

1. 파괴적인 필터링
   - 컬렉션을 순회하면서 선택된 원소를 제거해야 하는 경우
   - iterator의 remove를 사용해야 함
2. 변형
   - 리스트나 배열을 순회하면서 값 일부/전체를 교체해야 할 경우 반복자나 인덱스 사용이 필요함
3. 병렬 반복
   - 컬렉션을 병렬로 순회해야 할 경우 반복자와 인덱스 변수를 사용해 명시적으로 제어해야 함

- 위의 경우는 일반적인 for문을 사용해야 하므로 오류가 생기지 않도록 경계_해야 함

----

###### iterable 인터페이스 구현

- iterable을 구현한 객체라면 for each문을 사용할 수 있음
- 원소들의 묶음을 구현하는 타입을 작성한다면 iterable을 구현하는 것을 고민해볼 것