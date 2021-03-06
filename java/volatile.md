# volatile
## volatile 사용X
<center><img src="./../img/non-volatile1.png"></center>
<center><img src="./../img/non-volatile22.png"></center>

```java
public class SharedObject(){
    public int count = 0;
}
```
* 멀티코어 환경에서는 CPU마다 별도의 캐시를 가지고 있는데 만약 volatile을 사용하지 않는다면 다른 코어에서 실행되는 스레드가 별도의 캐시를 사용하므로 변수 값 불일치 문제가 발생할 수 있다.
## volatile 사용
<center><img src="./../img/volatile.png"></center>

```java
public class SharedObject(){
    public volatile int count = 0;
}
```
* volatile은 해당 변수를 메인 메모리에 저장하고 메인메모리로부터 값을 읽어 올 것을 명시하는 지정자이다.
* volatile을 사용하면 메인메모리에 값을 반영하고 읽어오는 것을 보장하기 때문에 변수 값 불일치 문제가 발생하지 않는다.
* volatile은 멀티스레드환경에서 하나의 스레드만 값을 write 하고 나머지 스레드가 read를 하는 상황에서 사용가능하다. 만약 여러 스레드가 공유 변수를 write하는 상황 이라면 volatile 만으로는 해결 불가능 하다.(synchronized, Atomic 변수 사용)