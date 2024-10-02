## 코틀린 블록
- run : 코루틴 블록 아님. 코드 가독성을 위한 블록
- runBlocking : suspend 함수를 동기적으로 실행하기 위한 블록
- launch : 비동기적으로 실행, 값 반환X
- async : 비동기적으로 실행, 값 반환
- withContext : 비동기적으로 실행, 값 반환, 코루틴을 메인 스레드가 아닌 다른 스레드에서 실행하고 싶을 때 사용
  * Dispatchers.Default -> 다른 워커스레드, cpu 작업
  * Dispatchers.IO -> IO스레드, io 작업
  * withContext 블록 하나당 하나의 스레드가 아니라 최소한의 스레드에서 최대한의 코루틴을 실행할 수 있도록 내부적으로 최적화되서 실행됨
- repeat : 지정된 횟수만큼 반복하여 블록 실행, 값 반환X
- supervisorScope : 부모-자식관계의 코루틴을 관리하는 블록, 블록 내부의 코루틴 중 하나가 실패해도 다른 자식 코루틴에 영향을 주지 않음
- coroutineScope : 부모-자식관계의 코루틴을 관리하는 블록, 블록 내부의 코루틴 중 하나가 실패하면 부모 코루틴(coroutineScope) 도 취소됨

- return@runBlocking : 이건 블록안에 블록이 있을 때만 필요함, 어떤 블록을 종료할지 명시하는거. 블록 안에 블록이 없더라도 return을 사용하고 싶으면 이렇게 블록을 명시해야함. return 사용 안하면 블록 마지막 값이 반환됨

## 코루틴 동작 방식
코루틴은 경량스레드라고 하는데 실제로 여러 스레드가 생성되는게 아님, 하나의 스레드에서 실행 포인터만 옮겨가는거고 스택도 스레드마다 하나만 있음. 코루틴의 상태정보는 코루틴마다 Continuation 객체가 생성되서 해당 객체에 상태정보가 저장되고 코루틴이 실행될 때 해당 객체에서 상태를 복원해서 실행하는거임

## 코루틴과 자바의 virtual thread 차이
코루틴과 가상스레드 : java platform thread는 커널 스레드이고(정확히는 사용자 스레드이지만 커널 스레드처럼 동작한다) virtual thread는 사용자 스레드이다. 코루틴은 스레드가 아니라 동일 스레드에서 실행흐름만 관리한다. 따라서 virtual 스레드는 컨텍스트 스위칭이 발생하고 코루틴은 발생하지 않기 때문에 속도면에서는 코루틴이 더 빠를 수 있다. 다만 코루틴은 하나의 스레드에서 실행되기 때문에(withContext 블록을 사용안하면) 병렬처리를 할 때는 virtual thread가 더 빠를 수 있다.

## 람다 블록
자바에서 Function 사용해서 람다를 파라미터로 전달하는 것 처럼 코틀린도 람다 블록을 파라미터로 전달가능함
```kotlin
val value = testService.doSomething(parameter = parameter) {
    //이 블록 내부의 로직이 doSomething의 block으로 전달됨
    getValue(request = request)
}
```
```kotlin
fun doSomething(parameter: Long, block: () -> String): String {
    println("doSomething 호출: parameter = $parameter")
    return block()  // 전달된 람다 블록을 즉시 호출
}
```
