## Effective Java 3/E

#### Item 37.  ordinal 인덱싱 대신 EnumMap을 사용하라

##### 잘못된 사용

```java
class Plant {
	enum LifeCycle { ANNUAL, PERENNIAL, BIENNIAL }
	
	final String name;
	final LifeCycle lifeCycle;
	
	public Plant(String name, LifeCycle lc) {
		this.name = name;
		this.lifeCycle = lc;
	}
	
	@Override
	public String toString() {
		return name;
	}
}

class PlanetClient{
	public static void main(String[] args) {
		@SuppressWarnings("unchecked")
		Set<Plant>[] plantsByLifeCycle = new Set[Plant.LifeCycle.values().length];
		Plant[] garden = {
			new Plant("A", Plant.LifeCycle.ANNUAL),
			new Plant("B", Plant.LifeCycle.BIENNIAL),
			new Plant("C", Plant.LifeCycle.PERENNIAL),
			new Plant("D", Plant.LifeCycle.PERENNIAL),
			new Plant("E", Plant.LifeCycle.BIENNIAL),
		};
		
		for(int i = 0; i< plantsByLifeCycle.length; i++) {
			plantsByLifeCycle[i] = new HashSet<>();
		}
		
		for(Plant p: garden) {
			plantsByLifeCycle[p.lifeCycle.ordinal()].add(p);
		}
		
		for(int i=0; i<plantsByLifeCycle.length; i++) {
			System.out.printf("%s: %s%n", Plant.LifeCycle.values()[i], plantsByLifeCycle[i]);
		}
	}
}

```

- 배열이나 리스트에서 원소를 꺼낼 때 ordinal 메서드로 인덱스를 얻는 경우가 있음
  → 잘못된 값이 들어와도 알기 어려움

------

##### EnumMap으로 수정한 코드

```java
		Map<Plant.LifeCycle, Set<Plant>> plantsByLifeCycle = new EnumMap<>(Plant.LifeCycle.class);
		
		for(Plant.LifeCycle lc : Plant.LifeCycle.values())
			plantsByLifeCycle.put(lc, new HashSet<>() );
		
		for(Plant p : garden)
			plantsByLifeCycle.get(p.lifeCycle).add(p);
		
		System.out.println(plantsByLifeCycle);

// EnumMap.Iterator
//public String toString() {
//    if (index < 0)
//        return super.toString();
//
//    return keyUniverse[index] + "="
//        + unmaskNull(vals[index]);
//}
```

- {ANNUAL=[A], PERENNIAL=[D, C], BIENNIAL=[B, E]} 같은 형식으로 예쁘게 출력됨
- 안전하고 성능도 배열과 비슷하기 때문에 EnumMap을 사용하는 것이 좋음

-----

##### Stream으로 최적화한 코드

```java
System.out.println(Arrays.stream(garden)
    .collect(Collectors.groupingBy(p -> p.lifeCycle, 
	    () -> new EnumMap<>(Plant.LifeCycle.class), Collectors.toSet() )));

static <T,K,D,A,M extends Map<K,D>> Collector<T,?,M>
  groupingBy(Function<? super T,? extends K> classifier, 
    Supplier<M> mapFactory, Collector<? super T,A,D> downstream)
```

- groupingBy로 p.lifeCycle을 키로 p를 매핑하고 EnumMap\<Plant.LifeCycle, Set>에 저장

  > toSet()은 특정한 알고리즘의 Set가 아닌 일반적인 Set의 collector를 반환
  > 특정한 Set가 필요하면 toCollection()을 사용할 것!

- Stream을 사용할 경우 해당 Key에 속하는 원소가 있을 경우에만 맵을 만들기 때문에 더 유리함

------

##### 결론

- 배열의 인덱스를 얻기 위해 ordinal 메서드를 사용하는 것은 삼가야 함
- 다차원 관계는 중첩 EnumMap을 사용할 것