## Effective Java 3/E

#### Item 60. 정확한 답이 필요하다면 float와 double은 피하라

- float와 double: 부동소수점 연산. 넓은 범위의 수를 빠르게 정밀한 근사치로 계산하도록 설계되어 있음
  → 금융 같은 **정확한 결과가 필요할 때는 사용해선 안됨**

------

###### 부동소수 타입을 사용한 연산

```java
public static void main(String[] args){
	double funds = 1.00;
    int itemsBought = 0;
    for(double price = 0.10; funds >= price; price += 0.10){
        funds -= price;
        itemsBought++;
    }
    System.out.println(itemsBought + "개 구입");
    System.out.println("잔돈(달러): "+fundss);
}
```

- out: 0.39999999...로 출력됨
- 정확한 결과가 필요할 때에는 BigDecimal, int, long 타입을 사용해야 함

###### BigDecimal 사용한 연산

```java
public static void main(String[] args){
	BigDecimal funds = new BigDecimal("1.00");
    int itemsBought = 0;
    for(double price = 0.10; funds >= price; price += 0.10){
        funds -= price;
        itemsBought++;
    }
    System.out.println(itemsBought + "개 구입");
    System.out.println("잔돈(달러): "+fundss);
}
```

- 기본 타입보다 쓰기가 불편하고 느림

###### int, long 사용한 연산

```java
public static void main(String[] args){
	int funds = 100;
    int itemsBought = 0;
    for(double price = 0.10; funds >= price; price += 0.10){
        funds -= price;
        itemsBought++;
    }
    System.out.println(itemsBought + "개 구입");
    System.out.println("잔돈(달러): "+fundss);
}
```

- 다룰 수 있는 값의 크기가 제한됨
- 소수점을 직접 관리해야하는 번거로움이 있음

-------

##### 결론

- 정확한 답이 필요한 경우에는 float나 double을 사용
- 코딩 시 불편함/성능 저하를 신경쓰고 싶지 않으면 → BigDecimal
- 성능이 중요하고 숫자가 너무 크지 않으면 → int, long