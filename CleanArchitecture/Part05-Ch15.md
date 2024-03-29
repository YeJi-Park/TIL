### Clean Architecture

#### 05부 아키텍처

#### 15장. 아키텍처란?

> **소프트웨어 아키텍트는 코드와 동떨어져서는 안된다.**

소프트웨어 아키텍트라면 코드에서 탈피하여 고수준의 문제에 집중해야 한다는 거짓말에 속아 넘어가서는 안됨
발생하는 문제를 경험해보지 않는다면, 다른 프로그래머를 지원하는 작업을 제대로 수행할 수 없기 때문

소프트웨어 시스템의 아키텍처란 **시스템을 구축했던 사람들이 만들어낸 시스템의 형태**
컴포넌트를 분할하는 방법, 분할된 컴포넌트를 배치하는 방법, 컴포넌트가 서로 의사소통하는 방식에 따라 정해짐
이 형태는 아키텍처 안에 담긴 소프트웨어 시스템이 쉽게 개발/배포/운영/유지보수되도록 만들어짐
→ 이를 용이하게 위해, **가능한 많은 선택지를 오래 남겨두는 전략을 따라야 함**

아마도 소프트웨어 아키텍처의 목표가 시스템을 제대로 동작하도록 만드는데 있다고 생각하고 있었겠지만, 시스템 아키텍처는 시스템의 동작 여부와는 거의 관련이 없음 
아키텍처는 운영보다는 배포,유지보수, 추가적인 개발 과정에 더 큰 영향을 미침
**아키텍처의 주된 목적은 시스템의 생명주기를 지원하는 것. 시스템의 수명과 관련된 비용은 최소화하고 프로그래머의 생산성은 최대화**
아키텍처가 시스템 동작에 아무 영향이 없다는 것이 아니라, 이 역할도 중요하지만 수동적이고 피상적인 것이기 때문에 본질적인 기능은 아님

-----

##### 개발

개발하기 힘든 시스템은 수명이 길지 않을 것이기 때문에시스템 아키텍처는 개발팀이 시스템을 쉽게 개발할 수 있도록 뒷받침해야 함

팀 구조가 다르면 아키텍처 관련 결정도 달라짐

- 소규모의 단일 개발팀: 잘 정의된 아키텍처가 없더라도 서로 협력하여 단일 시스템으로 개발할 수 있음
  개발 초기에는 오히려 아케틱처 관련 제약이 방해가 될 수 있음
- 대규모의 여러 개발팀: 잘 설계된 컴포넌트 단위로 분리하지 않으면 개발이 진척되지 않을 것
  다른 요소를 고려하지 않는다면 팀 별로 컴포넌트를 나누어 구성하게 될 테지만, 이런 구조는 시스템 생명주기에 최적은 아님

-----

##### 배포

소프트웨어 시스템의 배포 비용이 높을 수록 시스템의 유용성은 떨어짐
→ 아키텍처는 시스템을 단 한 번에 쉽게 배포할 수 있도록 만드는 데 목표가 있어야 함

초기 개발 단계에서는 배포 전략을 거의 고려하지 않아 개발은 쉽지만 배포하기는 어려운 아키텍처가 만들어짐

MSA를 사용할 경우, 컴포넌트가 잘 분리되고 인터페이스가 안정화되므로 개발에는 유리하지만 이 서비스들을 배포하는 데에 드는 비용이 커짐
아키텍트는 배포 문제를 초기에 고려해야 함

---

##### 운영

아키텍처가 시스템 운영에 미치는 영향은 다른 시스템 생명주기의 다른 단계보다는 적음
운영에서 겪는 대다수의 어려움은 하드웨어를 더 투입해서 해결할 수 있음

시스템 운영을 쉽게 해주는 아키텍처도 바람직하지만, 비용 관점에서 운영보다는 개발/배포/유지보수 쪽이 더 중요함

아키텍처는 시스템 운영에 필요한 도구를 알려줄 수 있음(?)
아키텍처는 시스템을 이해하기 쉽도록 유스케이스, 기능, 시스템의 필수 행위를 일급 엔티티로 격상시키고, 이들 요소가 개발자에게 주요 목표로 인식되도록 해야함

---

##### 유지보수

소프트웨어 시스템에서 비용이 가장 많이 드는 단계
어디를 수정하고, 어떤 전략을 쓸 지 결정하는 데 드는 비용과 이로 인한 side effect와 위험 부담 비용을 고려해야 함

시스템을 컴포넌트로 분리하고 안정된 인터페이스를 두어 격리하는 방식으로 아키텍처를 신중하게 만들면 이 비용을 줄일 수 있음

----

##### 선택사항 열어 두기

> **좋은 아키텍트는 결정되지 않은 사항의 수를 최대화 한다.**

소프트웨어를 부드럽게 유지하는 방법은 **세부사항(detail)에 대한 선택을 가능한 많이 오래 열어두는 것**

모든 소프트웨어 시스템은 두 가지 구성요소로 분해할 수 있음

- 정책: 모든 업무 규칙과 업무 절차를 구체화한 것
- 세부사항: 프로그래머가 정책과 소통할 때 필요한 요소지만, 정책의 행위에는 영향을 미치지 않음

아키텍트는 시스템에서 정책을 핵심적인 요소로 식별하고 세부사항은 정책에 무관하게 만들 수 있는 형태의 시스템을 구축해야 함
세부사항에 몰두하지 않은 채 고수준의 정책을 만들 수 있다면, 세부사항에 대한 결정을 연기할 수 있음
느리게 결정함으로써, 더 많은 정보를 가지고 제대로 된 결정을 내릴 수 있을 것

