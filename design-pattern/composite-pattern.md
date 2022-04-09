# 컴포지트 패턴
<img src="../img/composite-pattern.png">

```java
public class Client {
    Component composite1 = new Composite();
    Component composite2 = new Composite();
    Component leaf1 = new Leaf(3);
    Component leaf2 = new Leaf(1);
    Component leaf3 = new Leaf(5);
    Component leaf4 = new Leaf(7);
    composite2.add(leaf1);
    composite2.add(leaf2);
    // composite2 : {leaf1, leaf2}
    composite1.add(leaf3);
    composite1.add(leaf4);
    composite1.add(composite2);
    // composite1 : {composite2, leaf3, leaf4}
    System.out.println(composite1.operation());
}
```
```java
public interface Component {
    int operation();
}
```
```java
@Getter
public class Leaf implements Component {

    private int num;

    public Component(int num) {
        this.num = num;
    }

    @Override
    public int operation() {
        return getNum();
    }
}
```
```java
@Getter
public class Composite implements Component {

    private final List<Component> components = new ArrayList<>();

    @Override
    public int operation() {
        return components.stream().mapToInt(Component::operation).sum();
    }

    public void add(Component component) {
        components.add(component);
    }
}
```
* 컴포지트 패턴은 트리 구조를 통해 사용자가 단일 객체와 복합 객체를 동일하게 다루도록 한다. 즉, 복합 객체와 단일 객체는 같은 인터페이스를 구현하고, 복합 객체는 해당 인터페이스를 구현하는 객체들을 가지고 있다.
* 새로운 타입의 Leaf가 추가되더라도 Client의 코드를 바꿀 필요가 없다.