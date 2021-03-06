# 팩토리 메서드 패턴
팩토리 메서드 패턴은 객체의 생성과 처리 로직을 분리하여 결합도를 낮추는 디자인 패턴이다. 팩토리 메서드 패턴을 사용하면 처리 로직에 변화가 생겼을 때 변화가 생긴 처리 로직만 바꾸고 객체를 생성하는 코드는 손을 대지 않을 수 있다. 또한 새로운 ConcreteProduct를 추가해도 그에 해당하는 ConcreteCreator만 추가하면 되고 클라이언트의 코드에는 아무런 변화도 주지 않는다. 따라서 객체지향 프로그래밍의 OCP원칙을 만족한다. 팩토리 메서드 패턴은 객체를 생성하는데 템플릿 메서드 패턴을 적용했다고 보면 이해하기 쉽다.
<center><img src="./../img/factory-method.png"></center>

## 팩토리 메서드 패턴 적용 예시
```java
public interface ItemCreator {

    default Item create() {
        before();
        Item item = createItem();
        after();
        return item;
    }

    // 자바 9버전 이후부터 인터페이스에서 private 메서드 사용 가능
    // 자바 8버전 이하는 인터페이스가 아닌 추상 클래스 사용하자
    private void before(){ 
        System.out.println("Before Logic");
    }

    private void after(){
        System.out.println("After Logic");
    }

    Item createItem();
}
```
```java
public class ComputerCreator implements ItemCreator {

    @Override
    public Item createItem() {
        return new Computer();
    }
}
```
```java
public class CleanerCreator implements ItemCreator {

    @Override
    public Item createItem() {
        return new Cleaner();
    }
}
```
```java
public interface Item {
    public void use();
}
```
```java
public class Computer implements Item {

    @Override
    public void use(){
        System.out.println("use computer");
    }
}
```
```java
public class Cleaner implements Item {

    @Override
    public void use(){
        System.out.println("use cleaner");
    }
}
```
* 객체의 생성과 처리 로직을 분리했기 때문에 처리로직(use() 메서드)에 변화가 생긴다 하더라도 클라이언트가 객체를 생성하는 코드를 바꿀 필요가 없다.
* 새로운 ConcreteProduct가 추가되고 그에 해당하는 생성로직이 필요하다 하더라도 해당 객체를 생성하는 ConcreteCreator를 추가하기만 하면 된다.