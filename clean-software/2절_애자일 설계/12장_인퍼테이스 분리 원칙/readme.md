## ISP: 인터페이스 분리 원칙

### 클라이언트가 자신이 사용하지 않는 메소드에 의존하도록 강제되어서는 안된다.

## 예시: ATM 사용자 인터페이스

![isp](https://github.com/BaekGeunYoung/book-Agile-Software-development/blob/master/images/isp.PNG)

위 상황은 ISP가 지양하는 바로 그 상황이다. 각각의 Transaction 파생 클래스들은 모두 하나의 UI 인터페이스에 의존하게 되는데, 자신이 사용하지 않는 메소드에 의존하게 될 수 밖에 없으며 UI의 일부 메소드의 변경에 대한 영향이 이 UI에 의존하는 모든 Transaction 객체로 퍼져나가게 된다. 그래서 ISP를 적용한 올바른 형태는 아래와 같이 된다.

![isp2](https://github.com/BaekGeunYoung/book-Agile-Software-development/blob/master/images/isp2.PNG)

각 클라이언트는 자신이 직접 사용하는 메소드들에만 의존하고 있으므로 하위 모듈에서의 변경은 그 파급력을 최소화할 수 있다. 

> 새로운 Transaction 파생 클래스가 만들어질 때마다 그에 대응하는 인터페이스를 만들어야 하고, 이 변경사항은 맨 아래의 UI 클래스에 매번 반영되어야 하기 때문에 처음의 구조와 크게 다를 것이 없다고 생각할 수 있다. 하지만 아래의 구조는 위와는 달리 변경에 대한 파급력이 아래로 향하기 때문에 더 나은 구조라고 생각한다. LSP에서 말하듯 중요한 정책과 비즈니스 모델을 포함하는 상위 모듈을 독립시키는 것은 중요하며, 위 구조는 ISP를 지킴으로써 상위 모듈을 독립시키는 데 도움을 준다.
