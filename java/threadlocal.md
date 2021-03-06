# ThreadLocal
## 일반적인 변수 필드 공유
<center><img src="./../img/common-field1.png"></center>
<center><img src="./../img/common-field2.png"></center>

* 멀티 스레드 환경에서 일반적인 변수 필드를 사용해 값을 공유하면 처음 스레드가 보관한 데이터가 사라질 수 있다.

## ThreadLocal을 사용했을 때
<center><img src="./../img/threadlocal1.png"></center>
<center><img src="./../img/threadlocal2.png"></center>
<center><img src="./../img/threadlocal3.png"></center>

* ThreadLocal을 사용하면 ThreadLocal은 스레드마다 별도의 내부 저장소를 제공하여 다른 스레드가 같은 ThreadLocal 필드에 접근한다 해도 일반적인 변수 필드를 공유했을 때의 문제가 발생하지 않는다.

## ThreadLocal 사용 예시
```java
public class ThreadLocalEx {
 
    private ThreadLocal<String> sharedField = new ThreadLocal<>();

    public setSharedField(String s) {
        sharedField.set(s);
    }

    public getSharedField() {
        sharedField.get(s);
    }

    public removeSharedField() {
        sharedField.remove();
    }
}
```
* 여러 스레드가 하나의 ThreadLocalEx 객체를 공유한다해도 각 스레드 마다 별드의 String을 저장할 수 있다.
* set() : ThreadLocal에 값 저장
* get() : ThreadLocal의 값 조회
* remove() : ThreadLocal의 값 제거, __ThreadLocal을 다 사용하고나면 remove()를 호출해 ThreadLocal에 저장된 값을 제거해 주어야 한다.__
## ThreadLocal 사용시 주의사항
<center><img src="./../img/threadlocal-remove1.png"></center>
<center><img src="./../img/threadlocal-remove2.png"></center>

* spring의 경우에는 스레드가 필요할 때마다 생성하기에는 비용이 비싸기 때문에 스레드를 생성해 스레드풀에 저장해두고 필요하면 스레드풀에서 꺼내서 사용하고 전부 사용했다면 스레드 풀에 스레드를 반환한다.
* 스레드풀에서 스레드를 계속 재활용해서 사용하기 때문에 만약 다른 사용자가 ThreadLocal값을 사용한 후 remove()를 호출하지 않는다면 해당 사용자가 저장해두었던 값이 그대로 ThreadLocal에 남게된다.
* 만약 ThreadLocal에 저장된 값이 사용자의 개인정보와 같은 보안이 필요한 정보라면 큰 문제가 될 것이다.