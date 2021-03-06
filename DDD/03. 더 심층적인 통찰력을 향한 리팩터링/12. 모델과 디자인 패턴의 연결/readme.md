지금껏 연구되어 왔던 수많은 디자인 패턴들은 기술적인 측면에 좀 더 초점을 맞춘다. 하지만 그 중 일부는 도메인에서 마주치는 의미 있는 개념에 사용할 수 있다. 이번 장에서는 개념적인 차원에서 도메인 주도 설계 과정에 적용할 수 있는 디자인 패턴을 소개한다.

## STRATEGY (전략, 혹은 POLICY)

프로세스 모델링 작업을 하다보면 대상 프로세스를 적절하게 모델링하는 방법이 단 하나가 아니라는 사실을 깨닫곤 한다. 여러 대안 중 단 하나를 선택해버리면 프로세스의 정의가 어색해지고 복잡해질 수 있기 때문에, 우리는 중심이 되는 부분과 변화할 수 있는 부분을 분리하고 변화할 수 있는 부분은 상황에 따라 유동적으로 선택할 수 있도록 해야한다. 앞서 말했듯이 디자인 패턴은 기술적인 문제의 해결에 초점을 두고 있지만, 이 STRATEGY 패턴이 도메인 주도 설계와 결합했을 때는 '여러 프로세스의 선택'이라는 개념적인 문제를 해결할 수 있다.

프로세스에서 변화하는 부분을 별도의 전략(STRATEGY) 객체로 분리해서 모델에 표현하라. 프로세스의 규칙과 프로세스를 제어하는 행위를 서로 분리하라. 다양한 방식으로 변형된 전략 객체는 프로세스의 서로 다른 처리 방식을 표현한다. 

### 예제

STRATEGY 패턴은 개념 설명만으로는 의미가 잘 와닿지 않을 수 있다고 생각해 예제도 기록한다.

항로 운항 애플리케이션을 제작한다고 가정하자. (사실 본 책의 예제에서 가장 많이 쓰이는 가정이다.) Routing Service가 만들어내는 Itinerary(운항일정)은 다음 두 가지 조건 중 하나를 만족한다.

- 가장 빠르게 목적지에 도달하는 운항 일정
- 가장 저렴한 비용으로 목적지에 도착하는 운항 일정

Routing Service에는 지금 선택한 전략이 무엇이냐에 따라 분기하는 로직이 포함되어 있을 것이고, 우리는 Routing Service를 호출할 때 매개변수로 우리가 채택한 전략을 넘겨주면 된다. 이 예제에서 전략 객체는 구체적으로 Leg Magnitude Policy(구간 등급 정책)라는 이름을 가지고, 이 인터페이스를 Leg Time Magnitude, Leg Money Magnitude 객체가 각각 구현함으로써 STRATGEGY 패턴을 구현하게 된다.

## COMPOSITE

복잡한 도메인을 모델링하는 동안 종종 중요한 객체가 여러 개의 작은 부분으로 조합되어 구성돼 있는 경우가 있다. 그 중에는 특히 전체를 구성하는 부분이 단지 크기만 더 작을 뿐 전체와 완전히 동일한 종류의 것인 경우도 있다. 이러한 경우 복합 객체 간의 관련성을 모델에 반영하지 않을 경우 객체를 중첩할 수 있는 수준의 수는 제한될 것이며, 따라서 모델은 유연성을 상실한다.

따라서 적용하려는 디자인 패턴의 개념이 정말로 도메인 영역에 부합한다면(부분/전체 구조가 정말로 성립한다면) 재귀적인 구조를 사용해 도메인 영역의 지식을 모델에 반영해야 한다. 
