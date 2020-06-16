## 상속과 위임

이 장은 상속과 위임의 차이를 전형적으로 보여주는 두 개의 패턴에 대해 다룬다. TEMPLATE METHOD 및 STRATEGY 패턴은 비슷한 문제를 해결하고 보통 호환되어 쓰인다. 그러나 TEMPLATE MEHOD는 문제를 해결하기 위해 상속을 사용하는 반면, STRATEGY는 위임을 사용한다.

## TEMPLATE METHOD

화씨온도를 섭씨온도로 변환하는 프로그램의 예를 살펴보자. TEMPLATE METHOD 패턴을 사용하지 않고 단순하게 짜보면 아래와 같은 모양새가 될 것이다.

```java
public class ftocraw {
  public static void main() {
    InputStreamReader isr = new InputStreamReader(System.in);
    BufferedReader br = new BufferedReader(isr);
    boolean done = false;
    while (!done) {
      String fahrString = br.readLine();
      if (fahrString == null || fahrString.length() == 0)
        done = true;
      else {
        double fahr = Double.parseDouble(fahrString);
        double celcius = 5.0 / 9.0 * (fahr - 32);
        System.out.println("F=" + fahr + ", C=" + celcius);
      }
    }
    
    System.out.println("ftoc exit");
  }
}
```

TEMPLATE METHOD를 적용하기 위해서는 이 프로그램에서 일반적인 알고리즘을 분리해 인터페이스로 만들고, 이를 구현하는 파생 클래스를 만들어 구체적인 부분을 구현하도록 한다.

```java
pubilc abstract class Application {
  private boolean isdone = false;
  protected abstract void init();
  protected abstract void idle();
  protected abstract void cleanup();
  protected void setDone() {
    isDone = true;
  }
  protected boolean done() {
    return isDone;
  }
  
  public final void run() {
    init();
    while(!done())
      idle();
    cleanup();
  }
}

public class ftoctemplateMethod extends Application {
  private InputStreamReader isr;
  private BufferedReader br;
  
  protected void init() {
    isr = new InputStreamReader(System.in);
    br = new BufferedReader(isr);
  }
  
  protected void idle() {
    String fahrString = br.readLine();
    if (fahrString == null || fahrString.length() == 0)
      done = true;
    else {
      double fahr = Double.parseDouble(fahrString);
      double celcius = 5.0 / 9.0 * (fahr - 32);
      System.out.println("F=" + fahr + ", C=" + celcius);
    }
  }
  
  protected void cleanup() {
    System.out.println("ftoc exit");
  }
}
```

> 명심해야 할 것은 template method 패턴을 "왜" 쓰는가이다. template method 패턴은 일반적인 알고리즘, 즉 어플리케이션에서 널리 재사용될 가능성이 있는 고수준의 정책을 구체적인 구현으로부터 분리시키기 위해 쓰는 것이다. 위 예시에서도 Application이라는 추상 인터페이스를 만들면서 분리시킨 것은 init -> idle -> cleanup 으로 이어지는 재사용 가능한 상위 수준의 추상화이다. 다만 그 방법으로 뒤이어 설명할 strategy 패턴과는 다르게 상속을 택하고 있는 것이다.

## STRATEGY 패턴

strategy 패턴은 template 메소드와 사용하는 목적은 같지만, 그 목적을 달성하기 위해서 위임이라는 방식을 활용한다. template 메소드에서는 분리시키고 싶은 일반적인 알고리즘을 기반 클래스에 두었지만, strategy 패턴에서는 **일반적인 알고리즘**을 **구체적 클래스**인 ApplicationRunner에 두고, application 객체를 주입받아 일반적인 알고리즘을 실행하는 것에 대한 권한을 위임한다.

```java
public class ApplicationRunner {
  private Application itsApplication;
  public ApplicationRunner(Application app) {
    itsApplication = app;
  }
  public void run() {
    isApplication.init();
    while(!itsApplication.done())
      itsApplication.idle();
    itsAppliation.cleanup();
  }
}
```

strategy 패턴의 경우 일반적인 알고리즘과 구체적인 알고리즘 사이의 결합도를 조금 더 낮춤으로써 객체 간 책임을 명확히 할 수 있고, 유지보수에 도움이 될 수 있다. 
