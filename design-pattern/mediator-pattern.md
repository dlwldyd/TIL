# 중재자 패턴
<img src="./../img/Mediator_design_pattern.png">

```java
public class Client {

    private final Mediator mediator;

    private final int clientNum;

    public Client(Mediator mediator, int clientNum) {
        this.mediator = mediator;
        this.clientNum = clientNum;
    }

    public void print() {
        mediator.setClientNum(clientNum);
        mediator.setColleague(new ConcreteColleagueA(mediator));
        String colleagueA = mediator.execute();
        System.out.println(colleagueA);
        mediator.setColleague(new ConcreteColleagueB(mediator));
        String colleagueB = mediator.execute();
        System.out.println(colleagueB);
    }
}
```
```java
public interface Mediator {
    String execute();

    int getClientNum();

    void setColleague(Colleague colleague);

    void setClientNum(int clientNum);
}
```
```java
public class ConcreteMediator implements Mediator{

    private Colleague colleague;
    private int clientNum;

    @Override
    public void setColleague(Colleague colleague) {
        this.colleague = colleague;
    }

    @Override
    public void setClientNum(int clientNum) {
        this.clientNum = clientNum;
    }

    @Override
    public String execute() {
        return colleague.execute();
    }

    @Override
    public int getClientNum() {
        return clientNum;
    }
}
```
```java
public interface Colleague {
    String execute();
}
```
```java
public class ConcreteColleagueA implements Colleague{

    private final Mediator mediator;

    public ConcreteColleagueA(Mediator mediator) {
        this.mediator = mediator;
    }

    @Override
    public String execute() {
        return "ConcreteColleagueA" + mediator.getClientNum();
    }
}
```
```java
public class ConcreteColleagueB implements Colleague {

    private final Mediator mediator;

    public ConcreteColleagueB(Mediator mediator) {
        this.mediator = mediator;
    }

    @Override
    public String execute() {
        return "ConcreteColleagueB" + mediator.getClientNum();
    }
}
```
* Mediator가 없을 때는 클라이언트가 ConcreteColleague 들의 execute() 메서드를 실행하기 위해서 ConcreteColleague에 강하게 결합되지만 중간에 Mediator를 둠으로써 클라이언트는 Mediator에만 의존하게된다. 따라서 모든 ConcreteColleague 들에 대한 의존성은 Mediator에만 존재하게 되고 ConcreteColleague 들의 코드에 변화가 생긴다 해도 클라이언트 코드를 바꿀 필요가 없다.
* 다만 모든 의존성이 Mediator 한 곳으로 모이기 때문에 Mediator를 변경하기가 어렵다.