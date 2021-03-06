# 데코레이터 패턴
데코레이터 패턴은 실제 기능을 수행하는 객체와 클라이언트 사이에 프록시를 두어 부가기능을 추가하는 디자인 패턴이다. 프록시 패턴과 구조는 같지만 데코레이터 패턴과 프록시 패턴을 구분하는 차이는 해당 패턴을 접근제어를 위해 사용했는지 부가기능을 추가하기위해 사용했는지의 차이이다. 데코레이터 패턴을 사용함으로써 실제 기능을 수행하는 객체의 코드를 전혀 바꾸지 않고  부가기능을 추가할 수 있다.
<center><img src="./../img/decorator-pattern.png"></center>

## 데코레이터 패턴 적용 예시
### 인터페이스 기반
```java
@Slf4j
public class DecoratorPatternClient {

    private Component component;

    public DecoratorPatternClient(Component component) {
        this.component = component;
    }

    public void execute() {
        String result = component.operation();
        log.info("result={}", result);
    }
}
```
```java
public interface Component {
    String operation();
}
```
```java
@Slf4j
public class MessageDecorator implements Component{

    private Component component;

    public MessageDecorator(Component component) {
        this.component = component;
    }

    @Override
    public String operation() {
        log.info("MessageDecorator 실행");

        String result = component.operation();
        String decoResult = "***** " + result + " *****";
        log.info("MessageDecorator 꾸미기 적용 전={}, 적용 후={}", result, decoResult);
        return decoResult;
    }
}
```
```java
@Slf4j
public class RealComponent implements Component{
    @Override
    public String operation() {
        log.info("RealComponent 실행");
        return "data";
    }
}
```
```java
Component realComponent = new RealComponent();
Component messageDecorator = new MessageDecorator(realComponent);
DecoratorPatternClient client = new DecoratorPatternClient(messageDecorator);
client.execute();
```
* 실제 기능을 수행하는 객체와 프록시(데코레이터)는 같은 인터페이스를 상속받아야 한다.
* 프록시(데코레이터)를 여러개를 만들어서 chain의 형태로 여러개의 프록시(데코레이터)를 적용할 수 있다.
* 인터페이스를 기반으로 만들면 프록시(데코레이터)를 같은 인터페이스를 상속하는 모든 구체 객체에 대하여 적용시킬 수 있다.
* 프록시(데코레이터)들의 부모 클래스가 되는 추상클래스를 둬서 실제 기능을 수행하는 객체와 프록시(데코레이터)들을 분리할 수도 있다.
### 구체클래스 기반
```java
public class ConcreteClient {

    private ConcreteLogic concreteLogic;

    public ConcreteClient(ConcreteLogic concreteLogic) {
        this.concreteLogic = concreteLogic;
    }

    public void execute() {
        concreteLogic.operation();
    }
}
```
```java
@Slf4j
public class ConcreteLogic {

    public String operation() {
        log.info("ConcreteLogic 실행");
        return "data";
    }
}
```
```java
@Slf4j
public class TimeProxy extends ConcreteLogic {

    private ConcreteLogic concreteLogic;

    public TimeProxy(ConcreteLogic concreteLogic) {
        this.concreteLogic = concreteLogic;
    }

    @Override
    public String operation() {
        log.info("TimeDecorator 실행");
        long startTime = System.currentTimeMillis();

        String result = concreteLogic.operation();

        long endTime = System.currentTimeMillis();
        long resultTime = endTime - startTime;
        log.info("TimeDecorator 종료 resultTime={}ms", resultTime);
        return result;

    }
}
```
* 인터페이스를 만들지 않고 구체클래스를 기반으로 데코레이터 패턴을 적용할 수도 있다.
* 구체클래스를 기반으로 데코레이터 패턴을 적용한다면 해당 구체클래스에만 프록시(데코레이터)를 적용할 수 있다.
* 부모클래스에 final이 붙은 멤버변수가 있으면 부모클래스의 멤버를 사용하지 않더라도 부모클래스의 생성자를 호출해야한다.
* 프록시(데코레이터)를 적용하기 원하는 구체클래스들이 같은 인터페이스를 상속받지 않는다면 구체클래스를 기반으로 데코레이터 패턴을 적용할 수도 있다. 하지만 일반적으로는 인터페이스 기반으로 데코레이터 패턴을 적용하는 것이 더 좋다.