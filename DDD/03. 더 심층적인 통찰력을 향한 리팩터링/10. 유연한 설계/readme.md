소프트웨어는 1차적으로 사용자를 위한 것이지만, 다른 의미에서 개발자를 위한 것이기도 하다. 개발이 진행될수록 현재의 레거시 코드로 인한 중압감에 시달리지 않고 프로젝트 진행을 촉진하려면 변경을 수용하고 즐겁게 작업할 수 있는 설계가 필요하다. 이것이 바로 유연한 설계(supple design)이다. 심층 모델을 향한 리팩토링은 끊임없이 이루어져야 하며, 이 리팩토링이라는 작업을 쉽고 즐겁게 할 수 있어야 한다.

## INTENTION-REVEALING INTERFACE (의도를 드러내는 인터페이스)

캡슐화는 객체 지향 프로그래밍에서 매우 중요한 개념이다. 어떠한 클래스 혹은 메소드의 세부적인 구현사항을 알 필요 없이 겉만을 보고 그 대상으로부터 기대되는 결과를 명확히 추측할 수 있어야 한다. 심층 모델을 향한 리팩토링 과정에서도 영향 범위 내에 있는 메소드의 세부 구현사항과 타 메소드와 복잡하게 얽힌 연관관계를 모두 추적해야 한다면 리팩토링은 험난한 과정이 될 것이다.

**수행 방법에 관해서는 언급하지 말고 결과와 목적만을 표현하도록 클래스와 연산의 이름을 부여하라.** 이렇게 하면 클라이언트 개발자가 내부를 이해해야 할 필요성이 줄어든다. 이름은 팀원들이 그 의미를 쉽게 추측할 수 있게 ubiquitous language에 포함된 용어를 따라야 한다. 클라이언트 개발자의 관점에서 생각하기 위해 클래스와 연산을 추가하기 전에 행위에 대한 테스트를 먼저 작성하라.

## SIDE-EFFECT-FREE FUNCTION (부수효과가 없는 함수)

부수효과를 일으키지 않으면서 결과를 반환하는 연산을 함수라고 한다. 함수는 부수효과를 지닌 연산에 비해 테스트하기 쉬우며, 이러한 이유로 함수는 연산을 사용하는 데 따르는 위험을 낮춘다.

아래는 의도하지 않은 부수 효과를 줄이는 두 가지 방법이다.

첫째, 명령(command)과 질의(query)를 업격하게 분리된 서로 다른 연산으로 유지한다. 질의는 시스템으로부터 정보를 얻어오는 연산을, 명령은 정보를 조작하는 등 시스템의 상태를 변경하는 연산을 뜻한다. 

둘째, 명령과 질의를 분리하는 대신 연산의 결과를 표현하는 새로운 value object를 생성해서 반환한다. value object는 불변 객체이며, 이것은 value object에 관련된 연산은 모두 부수효과가 없는 함수라는 뜻이다. 복잡한 로직은 value object로 옮겨서 부수효과를 통제하고, 명령 연산은 아주 단순한 연산으로 유지한다.

(예제: 페인트 혼합 애플리케이션에서 Paint Entity로부터 pigment Color(안료 색소)를 VO로 분리하고, 색상 혼합에 관한 로직을 VO에 부여)

## ASSERTION (단언)

명령과 질의를 엄격히 분리하고, 복잡한 질의 로직을 VO로 옮긴다고 해도 명령에 의한 시스템의 상태 변경은 불가피하며, 본질적으로 우리가 지양해야 할 대상이 아니다. 중요한 것은 '명령을 없애는 것'이 아니라 '의도치 않은 부수효과를 어떻게 통제하는가'라고 생각한다.

연산의 부수효과가 단지 구현에 의해서만 함축적으로 정의될 때 다수의 위임을 포함하는 설계는 인과관계로 혼란스러워진다. 프로그램을 이해하려면 분기 경로를 따라 실행 결로를 추적하는 수밖에 없다. 이렇게 되면 캡슐화의 가치가 사라지고, 추상화가 무의미해진다.

그러므로 연산의 사후조건과 클래스 및 AGGREGATE의 불변식을 명시해야 한다. 코드에 직접 ASSERTION을 명시할 수 없다면 자동화된 단위 테스트를 작성해서 ASSERTION의 내용을 표현하면 된다. 또한 적절한 문서나 다이어그램으로 ASSERTION을 서술할 수 있다.

개발자들이 의도된 ASSERTION을 추측할 수 있게 인도하고, 쉽게 배울 수 있고 모순된 코드를 작성하는 위험을 줄이는 응집도 높은 개념이 포함된 모델을 만들려고 노력해야 한다.

## CLOSURE OF OPERATION (연산의 닫힘)

연산의 닫힘이란 수학적인 개념으로, 두 실수를 곱한 결과가 실수일 때 우리는 실수를 '곱셈에 대해 닫혀있다'라고 표현한다. 연산의 닫힘이 가지는 의미는 다른 개념의 개입 없이 완전하게 연산을 정의할 수 있다는 것이다. 불필요한 의존성을 줄이고자 노력하는 우리에게 연산의 닫힘은 적절한 비유가 될 수 있다.

적절한 위치에 반환 타입과 인자 타입이 동일한 연산을 정의하라. 이렇게 정의된 연산은 해당 타입의 인스턴스 집합에 대해 닫혀 있다. 닫힌 연산은 부차적인 의존성 없이도 고수준의 인터페이스를 제공할 수 있다.

특히 이 CLOSUER OF OPERATION은 술어의 연산에도 적용되므로, 9장에서 학습한 SPECIFICATION이 이 개념의 도움을 받아 선언적인 형태로 발전할 수 있다.
