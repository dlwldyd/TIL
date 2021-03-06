# 옵저버 패턴
옵저버 패턴은 객체(subject)에 옵저버를 등록하여 객체(subject)에 변화가 있으면 객체(subject)에 등록되어 있는 옵저버에게 통지하는 디자인 패턴이다. 옵저버 패턴을 통해 객체(subject)와 옵저버는 느슨하게 결합하여 객체(subject)와 옵저버는 서로 상호작용을 하지만 서로에 대해서 잘 모르게된다. 이로인해 객체(subject)나 옵저버가 바뀌더라도 서로에게 영향을 미치지 않게 되고, 새로운 옵저버가 추가되더라도 객체(subject)를 전혀 변경할 필요가 없게 된다.
<center><img src="./../img/observer-pattern.png"></center>

```java
public interface Subject {
	public void registerObserver(Observer observer);
	public void removeObserver(Observer observer);
	public void notifyObservers();
}
```
```java
public class Item1 implements Subject {

    private List<Observer> observers = new ArrayList<>();
    private int price;

    public int getPrice() {
        return price;
    }

    @Override
    public void registerObserver(Observer observer) {
        observers.add(observer);
    }

    @Override
    public void removeObserver(Observer observer) {
        observers.remove(observer);
    }

    @Override
    public void notifyObservers() {
        for (Observer observer : observers) {
            observer.update(price);
        }
    }

    public void priceChanged() {
        notifyObservers();
    }

    public void setPrice(int price) {
        this.price = price;
        priceChanged();
    }
}
```
```java
public interface Observer {
	public void update(int price);
}
```
```java
public class Client1 implements Observer {
    private int price;
    private Subject subject;

    public Client1(Subject subject) {
        this.subject = subject;
        item.registerObserver(this);
    }

    @Override
    public void update(int price) {
        this.price = price;
        System.out.println("현재 가격은 " + price + "원 입니다.");
    }
}
```
* subject가 하나일 경우에는 인터페이스가 아닌 구체클래스로 해도 괜찮다.
* subject와 옵저버는 서로에 대해서 모른다.(인터페이스에 의존한다) 따라서 subject나 옵저버에 변화가 있어도 서로에게 영향을 미치지 않으며 옵저버 클래스를 자유롭게 추가해도 subject의 코드를 수정할 필요가 없다.