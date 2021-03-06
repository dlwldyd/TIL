# 싱글톤 패턴
싱글톤 패턴은 객체를 딱 하나만 생성하여 생성된 객체를 어디에서든지 접근하여 사용할 수 있도록 하는 패턴이다.
<center><img src="./../img/Singleton_UML_class_diagram.png" width="40%" height="40%"></center>

## 싱글톤 패턴 적용 예시
```java
public class SingletonService {
    //어플리케이션 실행 시 객체가 생성되기 때문에 만약 해당 객체를 사용하지 않는다면 메모리 낭비가 된다.
    private static final SingletonService instance = new SingletonService();

    //생성자를 private로 만들어서 새로운 객체 생성을 막는다.
    private SingletonService(){}

    public static SingletonService getInstance(){
        return instance;
    }
}
```
```java
public enum SingletonService {
    //위와 마찬가지로 어플리케이션 실행 시 객체가 생성됨
    //상속 불가능
    instance;
}
```
```java
//double checked lock
/*
getInstance() 호출 시점에 객체를 생성하기 때문에 메모리 낭비가 없고, 
객체를 생성하는 코드가 샐행될 때 SingletonService 클래스에 대한 다른 쓰레드의 접근을 막기 때문에 thread safe하다.
*/
public class SingletonService {
    private static volatile SingletonService instance;

    //생성자를 private로 만들어서 새로운 객체 생성을 막는다.
    private SingletonService(){}

    public static SingletonService getInstance() {
        if (instance == null) {
            synchronized (SingletonService.class) {
                if (instance == null){
                    instance = new SingletonService();
                }
            }
        }
        return instance;
    }
}
```
```java
//가장 무난한 방법
//getInstance()가 호출되면 SingletonServiceHolder가 생성되면서 SingletonService 객체도 생성된다.
public class SingletonService {

    //생성자를 private로 만들어서 새로운 객체 생성을 막는다.
    private SingletonService(){}

    private static class SingletonServiceHolder {
        private static final SingletonService instance = new SingletonService();
    }

    public static SingletonService getInstance() {
        return SingletonServiceHolder.instance;
    }
}
```
## 싱글톤 패턴을 사용하는 이유
1. 하나의 객체를 사용함으로 인해 메모리 낭비를 방지할 수 있다.
2. 싱글톤 객체를 통한 객체들 간의 데이터 공유가 쉽다.
3. 객체가 절대적으로 한개만 존재하는 것을 보증한다.
## 싱글톤 패턴 사용 시 문제점
1. 의존관계상 클라이언트가 구체 클래스에 의존한다. => DIP 위반
2. 클라이언트가 구체 클래스에 의존해서 OCP 원칙을 위반할 가능성이 높다.
3. private 생성자로 자식 클래스를 만들기 어려워 유연성이 떨어진다.
4. 멀티쓰레드 환경에서 컨트롤이 어렵다.
## 싱글톤 패턴 사용 시 주의점
* 객체 인스턴스를 하나만 생성해서 공유하는 싱글톤 방식은 여러 클라이언트가 하나의 같은 객체 인스턴스를 공유하기 때문에 싱글톤 객체는 상태를 유지(stateful)하도록 설계하면 안된다.
```java
public class StatefulService {
    private static final StatefulService instance = new StatefulService();

    //특정 클라이언트에 의존적인 필드
    private int price;

    private StatefulService(){}

    public static StatefulService getInstance(){
        return instance;
    }

    //해당 함수를 통해 클라이언트가 price를 변경할 수 있다.
    public void setPrice(int price){
        this.price = price;
    }

    int getPrice(){
        return price;
    }
}
```
* 싱글톤 패턴을 사용할 때는 무상태(stateless)로 설계해야 한다.
    + 특정 클라이언트에 의존적인 필드가 있으면 안된다.
    + 특정 클라이언트가 값을 변경할 수 있는 필드가 있으면 안된다.
    + 가급적 읽기만 가능해야 한다.
    + 필드 대신에 자바에서 공유되지 않는, 지역변수, 파라미터, ThreadLocal 등을 사용해야 한다.