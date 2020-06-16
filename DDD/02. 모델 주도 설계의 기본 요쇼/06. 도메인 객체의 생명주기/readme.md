도메인 객체를 관리하면서 신경써야 하는 문제는 크게 아래 2가지이다.

1. 도메인 객체의 생명주기 동안 무결성 유지하기
2. 생명주기 관리의 복잡성으로 모델이 난해해지는 것을 방지하기

1번 문제를 해결하기 위한 방법으로 Aggregate 패턴을 알아볼 것이고, 2번 문제를 해결하기 위한 방법으로는 Repository와 Factory 패턴에 대해 알아볼 것이다.

## AGGREGATE (집합체)
entity 혹은 value object는 서로 밀접한 연관관계를 가지고 있는 경우가 많으며, 이러한 객체들 사이에 적용되는 불변식을 지키는 것에 신경써야 한다. 데이터 변경의 단위로 다루는
연관 객체의 묶음을 Aggregate라고 하며, 각 aggregate 사이의 경계를 명확히 함으로써 도메인 객체를 무결성 있게 유지할 수 있다. Aggregate는 설계 방법론적인 측면의 개념이지만, 
관계형 데이터베이스의 스키마 설계와 굉장히 비슷한 내용을 담고 있다는 느낌이 들었다. aggregate에 대한 세부 규칙은 다음과 같다.

- aggregate에는 중심이 되는 root entity와 경계가 존재한다.
- root entity는 전역 식별성을 지니며 궁극적으로 불변식을 검사할 책임이 있다.
- root가 아닌 다른 entity는 지역 식별성을 지닌다.
- 한 aggregate의 경계 밖에서는 root entity를 제외한 aggregate 내부의 구성요소를 참조할 수 없다. root entity를 거쳐서 aggregate 내부 entity의 참조를 일시적으로 사용할 수는 있지만,
참조를 계속 보유하고 있어선 안된다. value object의 경우 root entity가 복사본을 내어줄 수 있다.
- aggregate 안의 객체는 다른 aggregate의 루트만 참조할 수 있다.
- aggregate 경계 안의 어떤 객체를 변경하더라도 전체 aggregate의 불변식은 모두 지켜져야 한다.

## FACTORY (팩토리)
어떤 객체 혹은 Aggregate 전체를 생성하는 일이 많은 연관관계를 동반하거나 내부 구조를 너무 많이 드러내는 경우 객체를 생성하는 역할만을 전문적으로 담당하는 factory에게
그 책임을 위임할 수 있다. 간단한 객체의 생성은 해당 객체 내에서 처리할 수 있겠지만, 복잡한 조립 연산을 모델 객체 내에서 직접 수행하면 클라이언트 설계가 지저분해지고 
Aggregate의 캡슐화를 위반한다. **복잡한 객체를 생성하는 일은 도메인 계층의 책임이지만, 모델을 표현하는 객체의 책임은 아니다.**

### factory의 위치 선정
#### factory method (in Aggregate Root)
Aggregate에 새 요소를 추가해야 하는 경우, 해당 aggregate의 root entity에 factory method를 만들 수 있다. Aggregate 인스턴스 생성에 대한 책임을 루트가 담당하고, 인스턴스가 생성될 때
불변식 유지에 대한 책임도 이 팩토리 메서드에 있게 될 것이다.

#### factory method (in Other Class)
생성된 객체를 소유하지는 않지만 객체를 만들어내는 것과 밀접한 관련이 있는 특정 객체에 factory method를 두는 방법도 생각할 수 있다. 한 객체의 데이터나 규칙이 
다른 객체를 생성하는 데 매우 크게 영향을 주는 경우 이 방법을 활용하면 좋을 듯 하다.

#### factory class
standalone factory라고도 하며, Aggregate 인스턴스의 생성에 대한 책임을 맡은 factory가 필요한데, root가 그 책임을 맡기에 적절한 장소가 아니라면 독립된 factory class를
정의하는 방법을 생각해볼 수 있다. 독립된 팩토리 클래스는 다른 모든 객체들로부터 aggregate의 생성에 대한 내부 구현 사항을 숨기고 싶을 때 활용할 수도 있다.
