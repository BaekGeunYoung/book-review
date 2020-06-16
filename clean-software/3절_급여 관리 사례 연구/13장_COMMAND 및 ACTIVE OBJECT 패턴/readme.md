## COMMAND 패턴

COMMAND 패턴은 서로 다른 일을 하는 객체 그룹이 하나의 레벨로 묶이길 원할 때 사용하는 패턴이다. COMMAND 패턴의 핵심은 매우 단순한 인터페이스 하나로 설명될 수 있다.

```java
public interface Command {
  public void do();
}
```

이것은 command 객체 안에 존재하는 do() 함수의 수준을 클래스 수준으로 격상시키는 패턴이기 때문에 OO 패러다임을 망가뜨리는 것이라고 생각될 수 있고, 그것은 어쩌면 사실일 수도 있다. 하지만 언제나 그렇듯 적절한 활용은 OO 패러다임에서 조금 벗어날 지라도 소프트웨어를 한 층 세련되게 만들어 줄 수 있을 것이다.

## 예시: Transaction

Command 패턴을 활용할 수 있는 적절한 예시는 이번 절에서 소개하게 될 급여 관리 사례에서 등장하는 Transaction 클래스이다. 급여 관리 사례를 연구하면서 많은 종류의 Transcation 파생 클래스를 보게 될텐데, 그 중 하나인 AddEmployeeTransaction의 예를 들어보면 이 클래스가 작동하는 구조는 다음과 같다.

![command](https://github.com/BaekGeunYoung/book-Agile-Software-development/blob/master/images/command.PNG)

AddEmployeeTransaction 클래스는 Transaction 기반 클래스를 상속하여 validate()와 execute() 함수를 그 목적에 맞게 작성한다.

### 물리적/시간적 분리

이런 방식을 이용할 때의 장점은 사용자에게서 데이터를 받는 코드와 그 데이터를 검증하고 그것으로 작업을 하는 코드, 그리고 업무 객체 자체를 극적으로 분리하는 데 있다. 위 예시의 Transaction 클래스는 데이터를 검증하고 작업하는, 즉 데이터베이스의 자료를 조작하는 부분을 독립적으로 분리함으로써 어플리케이션 내에 존재하는 업무 로직 전체가 획일적인 형식을 취하게 되고 높은 가독성을 얻게 된다.
