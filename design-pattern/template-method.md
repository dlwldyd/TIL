# 템플릿 메서드 패턴
템플릿 메서드 패턴은 이름 그대로 템플릿을 사용하는 방식이다. 템플릿이라는 틀에 변하지 않는 부분을 몰아둔다. 그리고 변하는 부분을 서브클래스로 캡슐화해 전체 일을 수행하는 구조는 바꾸지 않으면서 특정 단계에서 수행하는 내역을 바꾸는 패턴이다.
<center><img src="./../img/TemplateMethodPattern.png" width="20%" height="20%"></center>

## 템플릿 메서드 패턴 적용 예시
### 추상클래스
```java
@Slf4j
public abstract class AbstractTemplate {

    //비즈니스 로직의 시간을 측정하는 부가기능(변하지 않는 부분)
    public void execute() {
        long startTime = System.currentTimeMillis();
        //비즈니스 로직 실행(변하는 부분)
        call();
        //비즈니스 로직 종료
        long endTime = System.currentTimeMillis();
        long resultTime = endTime - startTime;
        log.info("resultTime={}", resultTime);
    }

    //비즈니스 로직
    protected abstract void call();
}
```
### 구현클래스
```java
@Slf4j
public class SubClassLogic1 extends AbstractTemplate {
    //비즈니스 로직
    @Override
    protected void call() {
        
        ...

    }
}
```
* 자식 클래스가 알고리즘의 전체 구조를 변경하지 않고, 특정 부분만 재정의할 수 있다. 
* 템플릿 메서드의 단점으로는 자식 클래스 입장에서는 부모 클래스의 기능을 전혀 사용하지 않지만 자식 클래스가 부모 클래스와 컴파일 시점에 강하게 결합되는 문제가 있다. 이것은 의존관계에 대한 문제이다.
* 이러한 의존관계로 인해 부모 클래스를 수정하면, 자식 클래스에도 영향을 줄 수 있다.