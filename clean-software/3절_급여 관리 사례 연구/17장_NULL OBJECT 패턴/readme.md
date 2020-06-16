```java
Employee e = DB.getEmployee("Bob");
if (e != null && e.isTimeToPay(today))
    e.paY();
```

우리는 위와 같은 null 체크에 대해 익숙하다. 종종 이런 식의 코드는 가독성을 떨어뜨리며 에러가 발생하기 쉽다.

이런 경우 NULL OBJECT 패턴을 사용하면 null 검사의 필요를 제거하고 코드를 단순화하는데 도움을 줄 수 있다.

## NULL OBJECT 패턴

NULL OBJECT는 클래스의 정적 변수로서 선언하며, 기술적으로는 클래스의 인스턴스가 맞지만 개념적으로는 '아무 일도' 하지 않는다.

```java
public interface Employee {
    public boolean isTimeToPay(Date paDate);
    public void pay();
    public static final Employee NULL = new Employee() {
        public boolean isTimeToPay(Date PayDate) {
            return false;
        }

        public void pay() {
        }
    }
}
```

위 Employee 객체는 아래와 같이 단순한 코드를 작성할 수 있게 돕는다.

```java
Employee e = DB.getEmployee("Bob");
if (e.isTimeToPay(today))
    e.paY();
```

NULL OBJECT 패턴은 어떤 경우에도 항상 getEmployee 함수가 Employee 클래스의 인스턴스를 반환한다는 것을 보장하며, 그에 따라 코드는 훌륭한 일관성을 가질 수 있게 된다. 또한 NULL OBJECT 패턴은 개념적으로 아무 일도 하지 않기 때문에 기존의 로직을 변경할 필요도 없다.

> NULL 처리와 관련해서는 소프트웨어 설계 분야에서뿐만이 아니라 프로그래밍 언어 자체에서도 많은 노력을 기울여왔던 것 같다. java에서는 효율적인 null check를 위해 Optional 객체를 도입하기도 했고, 최근에 많이 사용하는 typescript나 kotlin 등의 언어는 null safety를 보장하는 물음표 연산자(?)를 가지고 있다.
>
> 따라서 NULL OBJECT 패턴을 활용하게 될 상황이 자주 올지는 모르겠지만, 디자인 패턴을 배우는 궁극적인 목적은 단순히 패턴을 사용하기 위해서라기보다는 그 패턴의 기저에 깔려 있는 발전적인 접하기 위함이라고 생각한다.