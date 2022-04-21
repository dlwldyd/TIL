# 비지터 패턴
<img src="./../img/visitor-pattern.png">

## 비지터 패턴 적용 전
```java
public class Client {

    public void execute() {
        Device watch = new Watch();
        Rectangle rectangle = new Rectangle();
        rectangle.printTo(watch);
        Device phone = new Phone();
        Triangle triangle = new Triangle();
        triangle.printTo(phone);
    }
}
```
```java
public interface Device {
}
```
```java
public class Phone implements Device {
}
```
```java
public class Watch implements Device {
}
```
```java
public interface Shape {

    void printTo(Device device);
}
```
```java
public class Triangle implements Shape {

    @Override
    public void printTo(Device device) {
        if (device instanceof Phone) {
            System.out.println("print Triangle to Phone");
        } else if (device instanceof Watch) {
            System.out.println("print Triangle to Watch");
        }
    }
}
```
```java
public class Rectangle implements Shape {

    @Override
    public void printTo(Device device) {
        if (device instanceof Phone) {
            System.out.println("print Rectangle to Phone");
        } else if (device instanceof Watch) {
            System.out.println("print Rectangle to Watch");
        }
    }
}
```
* 비지터 패턴은 기존의 알고리즘을 방문자에게 위임하는 패턴이다. 위의 예시를 보면 Shape 구현체들이 printTo 라는 알고리즘을 실행하고 있다. 비지터 패턴은 printTo에 해당하는 알고리즘을 printTo의 파라미터(Visitor)인 Device 객체에게 위임한다.
* 기존에는 Shape의 구현체가 늘어났을 때는 기존의 코드를 건드릴 필요가 없지만 Device 구현체가 늘어나면 각각의 Shape 구현체의 printTo() 메서드를 수정해야한다.
## 비지터 패턴 적용 후
```java
public class Client {

    public void execute() {
        Device watch = new Watch();
        Rectangle rectangle = new Rectangle();
        watch.print(rectangle);
        Device phone = new Phone();
        Triangle triangle = new Triangle();
        phone.print(triangle);
    }
}
```
```java
//Visitor
public interface Device {

    void print(Triangle triangle);

    void print(Rectangle rectangle);
}
```
```java
//Concrete Visitor
public class Phone implements Device {

    @Override
    public void print(Triangle triangle) {
        System.out.println("print triangle to Phone");
    }

    @Override
    public void print(Rectangle rectangle) {
        System.out.println("print rectangle to Phone");
    }
}
```
```java
//Concrete Visitor
public class Watch implements Device {

    @Override
    public void print(Triangle triangle) {
        System.out.println("print triangle to Watch");
    }

    @Override
    public void print(Rectangle rectangle) {
        System.out.println("print rectangle to Watch");
    }
}
```
```java
//Element
public interface Shape {

    void accept(Device device);
}
```
```java
//Concrete Element
public class Triangle implements Shape {

    @Override
    public void accept(Device device) {
        device.print(this);
    }
}
```
```java
//Concrete Element
public class Rectangle implements Shape {

    @Override
    public void accept(Device device) {
        device.print(this);
    }
}
```
* Visitor 패턴을 사용하면 Device 구현체가 늘어났을 때는 기존 코드를 전혀 손댈 필요 없고, Shape 구현체가 늘어났을 때는 Device 인터페이스와 각각의 Device 구현체의 기존 메서드를 손댈 필요 없이 새로운 메서드만 추가해주면 된다.