# 브릿지 패턴
<img src="../../TIL/img/bridge-pattern.png">

```java
@RequiredArgsConstructor
public class Client {
    
    private final Abstraction abstraction;

    public void useFunction() {
        abstraction.function();
    }
}
```
## 추상층 분리 전
```java
public interface Abstraction {
    void function();
}
```
```java
@RequiredArgsConstructor
public class ConcreteAbstraction implements Abstraction{

    @Override
    public void function() {
        System.out.println("ConcreteImplementor 1");
        System.out.println("RefinedAbstraction 1");
    }
}
```
* 브릿지 패턴은 구현부에서 추상층을 분리하는 디자인 패턴이다. 추상층을 분리하기 전에는 만약 "ConcreteImplementor 2" 와 "RefinedAbstraction 1"를 출력하고 싶다면 "ConcreteImplementor 2" 와 "RefinedAbstraction 1"를 출력하는 구현체를 따로 만들어 줘야한다. 그리고 이러한 조합들이 많아지면 많아질 수록 많아진 조합들의 수만큼 구현체를 만들어야 할 것이다.("ConcreteImplementor n", "RefinedAbstraction m" 의 조합을 출력하고 싶다면 n*m개의 구현체를 만들어야한다.)
```java
public interface Abstraction {
    void function();
}
```
```java
@RequiredArgsConstructor
public class RefinedAbstraction1 implements Abstraction{

    private final Implementor implementor;

    @Override
    public void function() {
        implementor.implementation();
        System.out.println("RefinedAbstraction 1");
    }
}
```
```java
public interface Implementor {
    void implementation();
}
```
```java
public class ConcreteImplementor1 implements Implementor{
    @Override
    public void implementation() {
        System.out.println("ConcreteImplementor 1");
    }
}
```
* 구현부에서 추상층을 분리한다면 "ConcreteImplementor 2" 와 "RefinedAbstraction 1"를 출력하고 싶다면 "ConcreteImplementor 2"를 출력하는 구현체만 만들면 되기 때문에 만약 조합들의 수가 많아지더라도 많아진 조합 수 만큼 구현체를 만들 필요가 없다.(n*m개 -> n+m개)