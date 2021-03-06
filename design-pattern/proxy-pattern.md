# 프록시 패턴
프록시 패턴은 실제 기능을 수행하는 객체와 클라이언트 사이에 프록시를 두어 해당 객체에 대한 접근을 제어하는 디자인 패턴이다. 데코레이터 패턴과 구조는 같지만 데코레이터 패턴과 프록시 패턴을 구분하는 차이는 해당 패턴을 접근제어를 위해 사용했는지 부가기능을 추가하기위해 사용했는지의 차이이다. 프록시 패턴을 사용함으로써 실제 기능을 수행하는 객체의 코드를 전혀 바꾸지 않고 접근제어 기능을 수행할 수 있다.
<center><img src="./../img/proxy-pattern.png"></center>

## 프록시 패턴 적용 예시
### 인터페이스 기반
```java
public class ProxyPatternClient {

    private Subject subject;

    public ProxyPatternClient(Subject subject) {
        this.subject = subject;
    }

    public void execute() {
        subject.operation();
    }
}
```
```java
public interface Subject {
    String operation();
}
```
```java
@Slf4j
public class RealSubject implements Subject{
    @Override
    public String operation() {
        log.info("실제 객체 호출");
        sleep(1000);
        return "data";
    }

    private void sleep(int millis) {
        try {
            Thread.sleep(millis);
        } catch (InterruptedException e) {
            e.printStackTrace();
        }
    }
}
```
```java
@Slf4j
public class CacheProxy implements Subject {

    private Subject target;
    private String cacheValue;

    public CacheProxy(Subject target) {
        this.target = target;
    }

    @Override
    public String operation() {
        log.info("프록시 호출");
        if (cacheValue == null) {
            cacheValue = target.operation();
        }
        return cacheValue;
    }
}
```
```java
RealSubject realSubject = new RealSubject();
CacheProxy cacheProxy = new CacheProxy(realSubject);
ProxyPatternClient client = new ProxyPatternClient(cacheProxy);
```
* 실제 기능을 수행하는 객체와 프록시는 같은 인터페이스를 상속받아야 한다.
* 프록시를 여러개를 만들어서 chain의 형태로 여러개의 프록시를 적용할 수 있다.
* 인터페이스를 기반으로 만들면 프록시를 같은 인터페이스를 상속하는 모든 구체 객체에 대하여 적용시킬 수 있다.
### 구체 클래스 기반
```java
``java
public class ProxyPatternClient {

    private RealSubject subject;

    public ProxyPatternClient(RealSubject subject) {
        this.subject = subject;
    }

    public void execute() {
        subject.operation();
    }
}
```
```java
@Slf4j
public class RealSubject {
    
    public String operation() {
        log.info("실제 객체 호출");
        sleep(1000);
        return "data";
    }

    private void sleep(int millis) {
        try {
            Thread.sleep(millis);
        } catch (InterruptedException e) {
            e.printStackTrace();
        }
    }
}
```
```java
@Slf4j
public class CacheProxy extends RealSubject {

    private RealSubject target;
    private String cacheValue;

    public CacheProxy(RealSubject target) {
        this.target = target;
    }

    @Override
    public String operation() {
        log.info("프록시 호출");
        if (cacheValue == null) {
            cacheValue = target.operation();
        }
        return cacheValue;
    }
}
```
```java
RealSubject realSubject = new RealSubject();
CacheProxy cacheProxy = new CacheProxy(realSubject);
ProxyPatternClient client = new ProxyPatternClient(cacheProxy);
client.execute();
```
* 인터페이스를 만들지 않고 구체클래스를 기반으로 프록시 패턴을 적용할 수도 있다.
* 구체클래스를 기반으로 프록시 패턴을 적용한다면 해당 구체클래스에만 프록시를 적용할 수 있다.
* 부모클래스에 final이 붙은 멤버변수가 있으면 부모클래스의 멤버를 사용하지 않더라도 부모클래스의 생성자를 호출해야한다.
* 프록시를 적용하기 원하는 구체클래스들이 같은 인터페이스를 상속받지 않는다면 구체클래스를 기반으로 프록시 패턴을 적용할 수도 있다. 하지만 일반적으로는 인터페이스 기반으로 프록시 패턴을 적용하는 것이 더 좋다.