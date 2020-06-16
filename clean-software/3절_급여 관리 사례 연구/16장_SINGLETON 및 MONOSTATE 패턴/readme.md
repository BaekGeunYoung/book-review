## 클래스와 인스턴스의 관계

종종 클래스와 인스턴스는 1:N 관계를 가진다. 그리고 인스턴스는 애플리케이션보다 짧은 라이프사이클을 가지며 생성되고 소멸되기를 반복한다.

하지만 단 하나의 인스턴스만을 가져야 하는 클래스도 있다. 팩토리 클래스 혹은 매니저 클래스 등이 그 예시인데, 이러한 클래스들은 애플리케이션이 시작했을 때 단 하나의 인스턴스가 출현하여 애플리케이션이 끝날 때 함께 사라져야 한다. 이러한 클래스의 인스턴스가 한 어플리케이션 내에서 여러 개가 만들어진다면 팩토리로 만드는 객체들에 대한 사무적인 제어가 불가능해지며, 둘 이상의 관리자가 있으면 순차적으로 하려고 했던 동작은 동시에 일어나게 될 것이다.

## SINGLETON

SINGLETON 패턴은 아주 단순한 패턴으로, 보통 아래와 같은 방식으로 구현된다.

```java
public class Singleton {
    private static Singleton theInstance = null;
    private Singleton() {
        ...
    }
    public static Singleton Instance() {
        if (theInstance == null) {
            theInstance = new Singleton();
        }
        return theInstance;
    }   
}
```

### SINGLETON이 주는 이점

- 어떤 클래스에도 쉽게 SINGLETON 패턴을 적용할 수 있다.
- 아무 기반 클래스에서 SINGLETON인 클래스를 파생시킬 수 있다.
- lazy evaluation : 만약 SINGLETON이 사용되지 않는다면 생성조차 되지 않는다.

### SINGLETON의 비용

- 인스턴스의 소멸이 정의되어 있지 않다.
- 상속되지 않는다. (SINGLETON 객체를 상속하는 객체는 SINGLETON이 아니다.)
- 비효율성 (Instance를 호출할 때마다 if 문을 거쳐야 한다.)
- 비투명성 (SINGLETON 객체의 호출자는 instance 메소드를 호출해야 하기 때문에 사실 SINGLETON 패턴의 사용에 대해 알고 있음)

## MONOSTATE

SINGLETON 패턴은 클래스의 인스턴스를 오직 하나만 만드는 패턴이었다면, MONOSTATE 패턴은 어떤 클래스의 모든 인스턴스가 **하나인 것처럼** 동작하게 만드는 패턴이다. 이 패턴의 구현은 아래와 같이 이루어진다.

```java
public class Monostate {
    private static int itsX = 0;
    public Monostate() {
        ...
    } 
    public void setItsX(int x) {
        itsX = x;
    }
    public int getItsX() {
        return itsX;
    }
}
```

어떤 클래스에 존재하는 멤버 변수는 곧 클래스의 상태라고 할 수 있다. MONOSTATE 패턴은 객체의 멤버 변수로 static한 변수만을 두어 모든 인스턴스가 단 하나의 상태를 공유하도록 한다.

### SINGLETON VS MONOSTATE

명심해야할 두 패턴의 차이점은 '동작 대 구조'의 차이라는 것이다. SINGLETON은 구조적인 측면에서 단일성 있는 구조를 강제하는 반면, MONOSTATE는 클래스에 구조적인 제약을 부여하지는 않지만 단일성 있는 동작을 강제한다.

### MONOSTATE가 주는 이점

- 투명성 (MONOSTATE 객체의 사용자는 이 객체가 MONOSTATE 객체라는 것을 알 필요가 없다.)
- 파생 가능성 (MONOSTATE 객체의 파생 클래스는 MONOSTATE이다.)
- 다형성 (MONOSTATE의 메소드는 정적이 아니기 때문에 이 클래스의 파생 클래스는 다형성을 가질 수 있다.)
- 잘 정의된 생성과 소멸

### MONOSTATE의 비용

- 변환 불가 (MONOSTATE가 아닌 클래스를 상속하여 MONOSTATE를 만들 수 없음)
- 여러번의 생성과 소멸을 겪을 수 있는 만큼 비용을 수반한다.
- MONOSTATE의 변수는 MONOSTATE가 사용되지 않는다고 해도 공간을 차지함.

