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
## 스코프 함수

| **함수 이름** | **참조 방식** | **반환 값**            | **주요 사용 목적**                     |
|---------------|---------------|------------------------|----------------------------------------|
| **`let`**     | `it`          | 람다 블록의 결과 값    | Null 체크, 변수 범위 제한, 체이닝       |
| **`run`**     | `this`        | 람다 블록의 결과 값    | 객체에 특정 연산 수행 후 결과 반환     |
| **`apply`**   | `this`        | 객체 자신             | 객체 설정 및 초기화                   |
| **`also`**    | `it`          | 객체 자신             | 부수 작업(로깅, 디버깅, 검증)          |
| **`with`**    | `this`        | 람다 블록의 결과 값    | 이미 생성된 객체에 대해 여러 작업 수행 |


### 예시

```kotlin
// let 예시: Null 체크와 변수 범위 제한
val name: String? = "Kotlin"
name?.let {
    println("The length of the name is: ${it.length}")
}

// run 예시: 특정 작업을 수행하고 결과 반환
val person = Person("Alice", 25)
val isAdult = person.run {
    println("Name: $name")
    age >= 18  // 블록의 마지막 값이 반환됨
}
println(isAdult)  // 출력: true

// apply 예시: 객체 설정 및 초기화
val person = Person().apply {
    name = "Alice"
    age = 25
}
println(person)  // Person(name=Alice, age=25)

// also 예시: 부가 작업 수행 (로깅)
val person = Person("John", 30).also {
    println("Person created: $it")
}

// with 예시: 이미 생성된 객체에 여러 작업 수행
val person = Person("Alice", 25)
val result = with(person) {
    println("Name: $name")
    println("Current Age: $age")
    age + 5  // 마지막 값이 반환됨
}
println(result)  // 출력: 30
```

## inline, noinline, crossinline
- inline : 해당 키워드가 붙은 함수를 인라인으로 처리한다.(보통 함수를 호출하면 해당 함수로 실행흐름이 이동하고 함수 호출 스택도 쌓이지만 인라인으로 실행하면 함수를 호출위치에 직접 복사에서 실행한다.)
```kotlin
inline fun printMessage(message: String) {
    println(message)
}

fun main() {
    printMessage("Hello, World!")  // 인라인 함수 호출
}
```
- noinline : 인라인 함수가 여러 람다를 인자로 받을 때 특정 람다만 인라인되지 않도록 설정하는 키워드이다.
```kotlin
inline fun processData(a: Int, b: Int, noinline operation: (Int, Int) -> Int): Int {
    return operation(a, b)  // `operation`은 인라인되지 않고, 람다 객체로 호출됩니다.
}

fun main() {
    val result = processData(4, 5) { x, y -> x + y }
    println(result)  // 출력: 9
}
```
- crossinline : 인라인 함수 내에서 람다가 비지역 반환(non-local return)을 하지 못하도록 설정합니다. 코틀린의 인라인 함수는 기본적으로 비지역 반환을 허용합니다. 즉, 인라인 함수 내에서 return을 사용할 경우, 람다가 종료되는 것이 아니라 인라인 함수를 호출한 함수 자체가 종료될 수 있습니다.
```kotlin
inline fun runWithCrossInline(crossinline action: () -> Unit) {
    println("Before action")
    action()  // `crossinline`을 사용하여 비지역 반환 금지
    println("After action")
}

fun main() {
    runWithCrossInline {
        println("Inside action")
        // return  // 오류: `crossinline` 키워드가 적용된 람다 내에서는 `return`을 사용할 수 없음
    }
}
```
```kotlin
// 비지역 반환 예시
inline fun doSomething(action: () -> Unit) {
    action()  // 람다 호출
    println("This will not be printed if `return` is used in action")
}

fun main() {
    doSomething {
        println("Before return")
        return  // `main` 함수가 종료됨
    }
    println("This will not be printed")  // 실행되지 않음
}
```