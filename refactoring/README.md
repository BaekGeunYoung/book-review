# Refactoring (Martin Fowler)

내용 자체는 당연한 말들의 연속이었다. 하지만 그 당연한 것들이 모두 명확하게 머리에 들어있는 사람은 많지 않다. 모든 개발자들이 리팩토링에 대해 이것저것 떠들곤 하지만, 그들 중 대부분은 상당 부분 빈약한 근거에 의존한 막연한 원칙들에 따라 리팩토링을 하고 있을 것이다. 이 책은 개발자들로 하여금 리팩토링을 할 때 활용할 수 있을 만한 테크닉들을 명시적으로 알려주고, 그러한 테크닉들이 가져올 수 있는 이득을 명확하게 알려줌으로써 리팩토링의 질을 한 단계 높여줄 수 있다는 데에 의미가 있을 것 같다. 이 책을 읽으면서 신경써야 할 부분은 단순히 나열된 테크닉을 학습하는 것이 아니라 어떤 상황에서 그 테크닉을 사용할 수 있을 지 고민해보는 것이라고 생각한다.

## 리팩토링의 원리

### 리팩토링이란?

> 소프트웨어를 보다 쉽게 이해할 수 있고, 적은 비용으로 수정할 수 있도록 겉으로 보이는 동작의 변화 없이 내부 구조를 변경하는 것.

### 두 개의 모자

kent beck은 소프트웨어 개발의 과정에서 개발자는 두 개의 모자를 번갈아 가며 쓴다고 표현한다. 이 둘은 각각 기능 추가와 리팩토링을 뜻하는데, 개발자들은 보통 개발을 해나가면서 수시로 모자를 바꿔쓰게 될 것이다. 개발자들은 항상 자신이 현재 어떤 모자를 쓰고 있는지 알고 있어야 한다.

### 리팩토링을 해야 하는 이유

- 디자인의 개선
- 코드의 가독성 개선
- 디버깅 용이성 증가
- 결과적인 생산성 증가

### 언제 리팩토링을 해야 하는가?

리팩토링은 리팩토링 그 자체보다 리팩토링을 언제 할 지 결정하는 것이 더 중요하다고 생각한다. 릴리즈가 다가오는데 기능 추가 없이 리팩토링만을 하고 있을 수도 없는 노릇이고, 리팩토링을 하고 있다는 사실이 팀 내에 확실히 전파되지 않을 경우 더 큰 혼란을 초래할 수도 있을 것이다. 아래는 언제 리팩토링을 할 지 대략적으로 결정할 수 있는 몇가지 가이드라인이다.

**삼진 규칙**

어떤 것을 처음 할 때는 그냥 한다. 두 번째로 비슷한 어떤 것을 하게 되면 중복 때문에 주춤하지만 그냥 중복되도록 한다. 세번째로 비슷한 것을 하게 되면 그때 리팩토링을 한다.

**기능을 추가할 때 리팩토링을 하라**

기능을 추가하기 위해서는 기존 코드를 확인해야 한다. 새로운 기능을 추가할 때 이해할 필요가 있는 기존 코드들이 가독성이 낮거나 잘못 설계되어 있다면 이 때가 리팩토링을 할 적절한 시기가 될 수 있다.

**버그를 수정해야할 때 리팩토링을 하라**

리팩토링은 버그를 수정할 때 코드를 좀 더 쉽게 이해하기 위해서 사용된다. 버그 리포트는 리팩토링이 필요하다는 신호로도 볼 수 있는데, 그 이유는 버그가 있었다는 것을 몰랐을 정도로 코드가 명확하지 않았다는 뜻이기 때문이다.

**코드 리뷰를 할 때 리팩토링을 하라**

코드 리뷰를 할 때 리팩토링을 하는 것은 당연하다.

## 코드 속의 나쁜 냄새

리팩토링이 필요한 시점에 대한 생각을 냄새로 표현하는 것은 꽤나 유명한 은유이다. 로버트 C. 마틴의 저서  "클린 소프트웨어"에서 객체 지향 설계 원칙을 지키지 않았을 때 코드가 어떤 악취를 풍기게 되는지 설명한 것처럼, 본 책에서도 리팩토링이 필요한 신호로 볼 수 있는 여러 악취를 소개한다.

## 테스트 만들기

테스트는 개발 방법론에 관한 책을 읽을 때면 언제나 빠지지 않고 등장하는 주제인 것 같다. 리팩토링의 관점에서도 기존 코드가 언제나 테스트 가능한 상태에 있다는 사실은 리팩토링이 기존의 동작에는 아무런 영향을 미치지 않도록 하는 것에 커다란 도움을 준다.

## 리팩토링 테크닉 소개

### 메소드 정리 & 객체간의 기능 이동

객체 지향 프로그래밍에서 가장 중요한 것은 책임이다. 각각의 객체가 본인의 책임을 온전히 다하고 있는 것이야 진정한 객체 지향 프로그래밍이고, 구체적으로 객체의 책임은 그 객체가 가지고 있는 상태(필드)와 행동(메소드)에 의해 구현된다.

아무리 설계를 잘하려고 애써도, 도메인 모델이 복잡해짐에 따라 객체의 책임은 흐려질 수 밖에 없고, 변화하는 요구사항 속에서 오늘은 적절한 책임을 갖고 있는 객체가 내일은 적절하지 않게 될 수도 있다. 주기적인 메소드 정리와 객체의 책임에 대한 감시는 대표적인 리팩토링 테크닉 중 하나이다.

### 데이터 구성

소프트웨어에서는 다양한 데이터가 생성되고 소멸되기를 반복한다. 정돈되지 않은 데이터는 필연적으로 코드의 중복을 유발하고, 상태와 행동이 산발적으로 흩어지게 만든다. 은밀히 내재되어 있는 데이터 간의 관계는 명시적으로 드러낼 필요가 있으며, 이는 코드의 악취 중 하나인 Primitive Obsession과도 밀접한 연관을 갖는다.

### 조건문의 단순화

조건문은 코드의 흐름을 제어할 수 있는 수단이다. 따라서 복잡하고 정리되지 않은 조건문의 남용은 코드의 가독성을 심각하게 떨어뜨릴 수도 있다. 위와 비슷하게 복잡한 조건문 속에서 은밀하게 내재해 있는 도메인 지식을 뽑아 내는 것이 조건문을 단순화하는 리팩토링의 핵심이다.

객체 지향에서 가장 중요한 개념 중 하나인 Polymorphism을 고려하지 않고 사용된 조건문은 경직성의 악취를 풍기게 하고, 따라서 조건이 추가될 때마다 조건문이 들어간 코드를 수정해야 한다. Polymorphism을 잘 활용하면 상당 수의 조건문을 제거할 수 있고, 높은 확장성을 갖게 된다.

### 일반화 다루기

객체 지향의 Polymorphism과 이를 근간으로 하는 도메인 주도 설계 (DDD)에서 결국 궁극적으로 이루어 내고자 하는 것은 때와 상황에 의존적이지 않은, 보편적이고 일반적으로 적용할 수 있는 도메인 지식을 이끌어내는 것이다. 고도화된 일반성은 코드의 재사용성과 확장성을 높인다. 여러 도메인 객체들이 갖고 있는 공통적이고, 일반적인 속성을 추출해 super class로 만들거나 subclass로 만드는 등 객체들 간의 상속관계를 정립할 수 있다. 또한 관계를 갖긴 하지만 완벽한 is-a 관계가 아닐 시에는 상속 대신 위임을 활용할 수 있다. 상속과 위임은 여러 객체들이 공유하는 일반적인 특성을 추출해낼 수 있는 적극적인 전략이다.