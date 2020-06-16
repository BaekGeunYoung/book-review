### 하위 타입은 그것의 기반 타입에 대해 치환 가능해야 한다.

```c
void drawShape(Shape &s) {
  if (s.itsType == Shape :: square)
    static_cast < const Square & > (s).draw();
  else
    static_cast < const Circle & > (s).draw();
}
```

위 함수는 LSP를 위반한다. 현재는 Shape라는 기반 클래스를 상속하는 클래스가 Circle 클래스와 Square 클래스 뿐이지만, 이를 상속하는 또 다른 클래스 Triangle이 추가된다면 이 함수에 등장하는 Shape(기반 타입)은 Triangle(하위 타입)로 완전하게 치환될 수 없다.

이러한 함수 구조에서 Triangle 클래스를 추가하게 되면 drawShape 함수가 변경되어야 하므로 이는 OCP를 위반한 것이다. 즉, LSP의 위반은 OCP의 위반을 유발한다.

## LSP와 IS-A 관계

LSP는 분명히 클래스의 상속에 관한 설계 원칙이고, 어떤 클래스들이 상속 관계에 있는 지를 파악하기 위해서는 IS-A 관계를 따져보는 것이 도움이 된다. 위 예시를 다시 한 번 사용해보면 "Circle is a Shape"는 참인 명제이기 때문에, Circle은 Shape를 상속하기에 적절하다.

## 미묘한 LSP의 위반

책에서는 좀 더 미묘한 LSP의 위반를 보여주기 위해 Rectangle 클래스와 Square 클래스의 예시를 든다. 수학적으로 생각해 보았을 때, 모든 정사각형은 직사각형이므로 Square 클래스는 Rectangle 클래스의 하위 타입이 되기에 적절해 보인다. 우선 아래와 같이 Rectangle 클래스를 정의한다.

```java
class Rectangle {
  private double itsHeight;
  private double itsWidth;
  
  public void setWidth(double w) {
    itsWidth = w;
  }
  
  public void setHeight(double h) {
    itsHeight = h;
  }
  
  public getHeight() {
    return itsHeight;
  }
  
  public getWidth() {
    return itsWidth;
  }
}
```

그리고 이 클래스를 상속하는 Square 클래스를 정의해야 하는데, 이 때 생각해야 할 점은 정사각형은 직사각형과는 다르게 width와 height가 항상 같다. 그래서 직사각형과는 달리 itsHeight와 itsWidth라는 2개의 멤버 변수도 필요없다. 우리는 이러한 문제로부터 설계의 무언가가 잘못되었다는 것을 느끼기 시작하지만, 우선은 원래의 계획대로 Rectangle을 상속해 Square를 만들어 보기로 한다.

```java
class Square extends Rectangle {
  @Override
  public void setWidth(double w) {
    itsWidth = w;
    itsHeight = w;
  }
  
  @Override
  public void setHeight(double h) {
    itsWidth = h;
    itsHeight = h;
  }
}
```

언급했듯이 정사각형의 가로와 세로의 길이는 항상 같으므로 기반 클래스의 setWidth 함수와 setHeight 함수를 오버라이드하여 가로와 세로의 길이를 항상 같게 유지할 수 있도록 한다.

이렇게 하면 문제 없이 Rectangle과 Square에 대한 적절한 클래스가 정의된 것 같지만, 아래와 같은 함수에서 문제가 발생한다.

```java
void f(Rectangle &r) {
  r.setWidth(5);
  r.setHeight(4);
  assert(r.area() == 20);
}
```

이 함수는 인자로 Rectangle 클래스를 넘겨받는다면 정상적으로 동작하겠지만, Square클래스를 넘겨받는다면 에러를 뱉을 것이다. 이러한 문제가 발생하는 근복적인 이유는 "f의 제작자는 Rectangle의 가로 길이를 바꾸는 것이 세로 길이를 바꾸지는 않을 것이라고 생각한다"는 데에 있다. 이 함수는 기반 타입이 하위 타입으로 완벽히 치환될 수 없으므로, 명백한 LSP의 위반이다.

## IS-A는 동작에 대한

우리는 왜 이러한 오류를 범했는지 생각해보아야 한다. 우리는 처음 Rectangle과 Square를 설계할 때, 둘 사이에 IS-A 관계가 성립한다고 믿고, 이 둘을 상속 관계로 묶기로 결정했다. 하지만 위 예시에서 **f 함수의 제작자가 생각하는 한 Rectangle과 Square는 IS-A 관계가 성립하지 않는다!** IS-A 관계는 사실 객체 자체에 관한 것이 아니라, 객체의 동작에 관한 것이다. 도형의 가로와 세로 길이를 설정하는 동작에 관해서, 정사각형은 직사각형이 아닌 것이다.

## 계약에 의한 설계 (Design By Contract)

IS-A 관계가 절대적인 것이 아니라 소프트웨어에 종속적인 "객체의 동작"에 관한 것이라면, 우리는 어떻게 LSP를 엄격하게 지킬 수 있을까? 이는 객체의 동작에 관한 합리적인 가정을 명시적으로 만들어 LSP를 강제하도록 하는 계약에 의한 설계(DBC) 방법을 사용할 수 있다. 구체적으로 이는 메소드의 사전조건과 사후 조건을 명시함으로써 달성될 수 있는데, 이 명시는 문서화를 통해 이루어질 수도, 테스트 코드에서 이루어질 수도 있다.

> 설계 원칙에 대해 공부해 가면서 드는 생각은, 아무리 중요한 설계 원칙들도 절대적인 것은 아니며 제작하려는 소프트웨어의 상황과 특징을 고려하여 신중하게 원칙들을 적용해야 한다는 것이다. OCP를 맹신한 무분별한 추상화는 불필요한 복잡성을 낳을 수 있으며, 소프트웨어 특이성을 고려하지 않은 상속 관계는 LSP의 위반으로 이어질 수 있다. 원칙의 개념 자체도 중요하지만, 원칙을 적절히 적용할 수 있는 경험과 통찰 또한 매우 중요하다는 생각이 들었다.
