# 이터레이터 패턴
<img src="../img/iterator-pattern.png">

```java
//Client
public class App {

    public static void main(String[] args) {
        Aggregate aggregate = new ConcreteAggregate();
        aggregate.addItem("item");
        aggregate.addItem("itemm");
        aggregate.addItem("itm");
        aggregate.addItem("iteem");

        Iterator<Item> iterator = new ConcreteIterator(aggregate);
        iterator.forEachRemaining(item -> System.out.println(item.getItemName()));
    }
}
```
```java
public interface Aggregate {

    void addItem(String itemName);

    Iterator<Item> getConcreteIterator();
}
```
```java
public class ConcreteAggregate implements Aggregate{

    List<Item> items = new ArrayList<>();

    public List<Item> getItems() {
        return items;
    }

    @Override
    public void addItem(String itemName) {
        this.items.add(new Item(itemName));
    }

    @Override
    public Iterator<Item> getConcreteIterator() {
        return items.iterator();
    }
}
```
```java
//자바에서 제공하는 Iterator 인터페이스
public interface Iterator<E> {
   
    boolean hasNext();

    E next();

    default void remove() {
        throw new UnsupportedOperationException("remove");
    }

    default void forEachRemaining(Consumer<? super E> action) {
        Objects.requireNonNull(action);
        while (hasNext())
            action.accept(next());
    }
}
```
```java
//List<Item> 를 순회하는 ConcreteIterator
public class ConcreteIterator implements Iterator<Item> {

    private Iterator<Item> internalIterator;

    public ConcreteIterator(Aggregate aggregate) {
        this.internalIterator = ((ConcreteAggregate) aggregate)
                .getItems().stream()
                .filter(item ->
                        item.getItemName().contains("item"))
                .collect(Collectors.toList())
                .iterator();
    }

    @Override
    public boolean hasNext() {
        return internalIterator.hasNext();
    }

    @Override
    public Item next() {
        return internalIterator.next();
    }
}
```
* 이터레이터 패턴은 집합 객체의 내부구조를 노출시키지 않고 집합 객체를 순회할 수 있는 디자인 패턴이다.
* 집합 객체의 내부구조를 노출시키지 않기 때문에 집합 객체를 감싸고 있는 자료구조가 변한 다면(List -> Set 처럼) 클라이언트 코드를 수정하지 않고 해당 자료구조의 순회를 지원하는 ConcreteIterator 객체를 추가함으로써 객체의 순회를 가능하게 할 수 있다.
* 집합 객체를 감싸는 자료구조가 변할 가능성이 높을 경우에 이터레이터 패턴을 사용하는 것이 좋다.
* 자바에서는 기본적으로 Iterator 인터페이스를 제공하고 있기 때문에 이터레이터 패턴 사용 시 Iterator 인터페이스를 구현하여 손쉽게 이터레이터 패턴을 구현할 수 있다.