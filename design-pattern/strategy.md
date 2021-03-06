# 전략 패턴(템플릿 콜백 패턴)
전략 패턴은 변하지 않는 부분을 Context 라는 곳에 두고, 변하는 부분을 Strategy 라는 인터페이스를 만들고 해당 인터페이스를 구현하도록 해서 알고리즘을 캡슐화 하여 동적으로 알고리즘을 유연하게 변경할 수 있게 해주는 패턴이다.
<center><img src="./../img/strategyPattern.png"></center>

## 전략 패턴(템플릿 콜백 패턴) 적용 예시
```java
@Slf4j
@RequiredArgsConstructor
public class ContextV2 {

    //Context 생성 시점에 주입 받을 수 있다.
    private final Strategy strategy;

    //비즈니스 로직의 시간을 측정하는 부가기능(변하지 않는 부분)
    public void execute1() {
        long startTime = System.currentTimeMillis();
        //비즈니스 로직 실행
        strategy.call(); //위임
        //비즈니스 로직 종료
        long endTime = System.currentTimeMillis();
        long resultTime = endTime - startTime;
        log.info("resultTime={}", resultTime);
    }

    //execute 실행 시점에 비즈니스 로직을 정의할 수 있다.
    public void execute2(Strategy strategy) {
        long startTime = System.currentTimeMillis();
        //비즈니스 로직 실행(변하는 부분)
        strategy.call();
        //비즈니스 로직 종료
        long endTime = System.currentTimeMillis();
        long resultTime = endTime - startTime;
        log.info("resultTime={}", resultTime);
    }
}
```
```java
public interface Strategy {
    void call();
}
```
```java
//Context 생성 시점에 주입 받는다면 이러한 구현 클래스를 정의할 필요가 있지만 execute 실행시점에 매개변수로 받는다면 이러한 구현클래스를 만들어둘 필요가 없다.(익명 클래스)
public class StrategyLogic implements Strategy {

    @Override
    public void call() {
        
        ...

    }
}
```
* execute1()을 사용할 경우 전략(알고리즘)을 바꾸려면 새로운 Context객체를 생성해 주어야 하지만 execute2()를 사용한다면 새로 객체를 생성할 필요가 없다.
* 스프링에서 의존관계 주입이 이러한 템플릿 콜백 패턴(전략 패턴)의 일종이라 할 수 있다.(주입되는 객체를 바꿔 끼우기만 하면 알고리즘을 바꿀 수 있다.)
* 템플릿 메서드 패턴과 달리 Context 클래스를 수정하더라도 Strategy 인터페이스에만 의존하기 때문에 Strategy의 구현체를 일일이 수정할 필요가 없다. 만약 자식 클래스(변하는 부분)가 부모클래스(변하지 않는 부분)의 요소를 사용한다면 템플릿 메서드 패턴을 사용하는 것이 좋겠지만 그렇지 않다면 전략 패턴을 사용하는 것이 더 좋다.