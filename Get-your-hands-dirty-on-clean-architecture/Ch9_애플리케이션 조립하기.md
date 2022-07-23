## 만들면서 배우는 클린 아키텍처

### Ch09. 애플리케이션 조립하기

#### 왜 조립까지 신경써야 할까?

- 도메인 코드가 바깥 계층의 변경으로부터 안전하게 하기 위해서 의존성과 인스턴스를 생성하는 책임의 주체가 중요함
  💡 유스케이스는 인터페이스만 알아야 하고, 런타임에 구현을 제공받아야 함

![설정 컴포넌트](https://github.com/YeJi-Park/TIL/blob/main/Get-your-hands-dirty-on-clean-architecture/images/09_1.png)

- 모든 클래스에 의존성을 갖는 **설정 컴포넌트**에서 객체 인스턴스를 생성할 책임을 가지고 있어야 함
- 모든 내부 계층에 접근하는 원의 가장 바깥쪽에 위치하고, 어플리케이션의 제어를 위해 설정 파라미터에도 접근이 가능해야 함
- 단일 책임 원칙을 위반하는 것이 맞지만, 어플리케이션의 나머지 부분을 깔끔하게 유지하기 위한 희생!

#### 평범한 코드로 조립하기

```java
public class Application {
    public static void main(String[] args) {
        AccountRepository accountRepository = new AccountRepository();
        ActivityRepository activityRepository = new ActivityRepository();

        AccoutPersistenceAdapter accountPersistenceAdapter =
                new AccoutPersistenceAdapter(accountRepository, activityRepository);

        SendMoneyUseCase sendMoneyUseCase =
                new SendMoneyService(
                        accountPersistenceAdapter, // LoadAccountPort
                        accountPersistenceAdapter // UpdateAccountStatePort
                );

        SendMoneyController sendMoneyController =
                new SendMoneyController(sendMoneyUseCase);

        startProcessingWebRequests(sendMoneyController);
    }
}

```

- 가장 기본적인 방법이지만, 코드가 복잡해지면 수많은 컨트롤러 유스케이스가 연결되어야 하는 문제점이 있음
- 패키지 외부에서 인스턴스를 생성하기 때문에 클래스들이 전부 public이어야 함
  → 유스케이스가 영속성에 접근하는 것을 막을 수 없음

### 스프링의 클래스패스 스캐닝으로 조립하기

- 스프링에서 제공하는 의존성 주입 방법으로 @Component 애너테이션이 붙은 클래스를 찾아 객체를 생성해줌
- 커스텀 애너테이션을 생성해 클래스패스 스캐닝이 되도록 할 수도 있어 아키텍처 파악이 쉬워질 수 있음
- 단점
  - 프레임워크에 특화된 애너테이션을 붙여야 한다는 점에서 침투적임
  - 스프링과 엮이며 부수효과를 발생시킬 수 있음

#### 스프링의 자바 컨피그로 조립하기

- 애플리케이션 컨텍스트에 추가할 빈을 생성하는 설정 클래스를 만들고 @Configuration 애너테이션을 통해 이 클래스가 클래스패스 스캐닝에서 발견되어야 할 클래스임을 표시함
  🔔 여전히 클래스패스 스캐닝을 사용하지만 모든 클래스가 아니라, 설정 클래스만 선택해 가져오게 됨

- 필요로 하는 한정적인 범위만 인스턴스화할 수 있기 때문에 특정 모듈만 포함하고 다른 모듈은 모킹하는 방식으로 유연하게 테스트가 가능함

- @Component 애너테이션이 남용되지 않으므로 애플리케이션에 의존성 없이 깔끔하게 유지할 수 있음

- ❗ 설정 클래스와 다른 패키지에 있는 빈들은 public으로 선언되어야 하는 문제가 남아있음

  > 패키지를 모듈 경계로 사용하고 패키지 별로 전용 설정 클래스를 만들 수는 있음

#### 유지보수 가능한 소프트웨어를 만드는 데 어떻게 도움이 될까?

- 클래스패스 스캐닝을 사용하면 빠르게 개발할 수 있지만 투명성이 낮아질 수 있음
- 설정 컴포넌트를 만들면 책임에서 자유로울 수 있고 응집도가 높아질 수 있지만 설정 컴포넌트를 유지보수해야 하는 문제가 있음
- 프로젝트의 특성에 맞게 적절한 방법을 사용해야 함