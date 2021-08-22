## Effective Java 3/E

#### Item 27. 비검사 경고를 제거하라

- 비검사 경고: 런타임에 ClassCastException을 발생시킬 여지가 있음

------

###### @SuppressWarning ("unchecked")

- 타입 안전하다고 확신할 수 있다면 annotation 달아서 해결 가능
- **가능한 좁은 범위에 사용해야 함**
- annotation은 선언에만 사용 가능하므로 새로운 변수를 선언해서라도 지역변수와 같은 좁은 범위로 이동시켜야 함
- 다른 사람이 코드를 수정할 때에 타입 안정성을 잃지 않을 수 있도록, annotation의 이유를 주석으로 달아야 함

