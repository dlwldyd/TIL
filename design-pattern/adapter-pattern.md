# 어댑터 패턴
<img src="../../TIL/img/adapter-pattern.png">

```java
//기존 코드
@RequiredArgsConstructor
public class Client {
    
    private final Target target;

    public void useRequest() {
        target.request();
    }
}
```
```java
//기존 코드
public interface Target {
    void request();
}
```
```java
//추가하는 코드
@RequiredArgsConstructor
public class Adapter implements Target {

    private final Adaptee adaptee;

    @Override
    public void request(){
        adaptee.specificRequest();
    }
}
```
```java
//기존 코드
public class Adaptee {
    
    public void specificRequest {
        System.out.println("adpater pattern");
    }
}
```
* 클라이언트가 사용하는 인터페이스와 Adaptee가 사용하는 인터페이스가 전혀 다를 때 사용하는 디자인 패턴이다.
* Adapter를 추가함으로써 Client와 Adaptee의 코드를 전혀 바꾸지 않고 Client가 Adaptee를 사용할 수 있도록 해준다.