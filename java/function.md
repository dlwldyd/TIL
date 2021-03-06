# java.util.function
## Function
<center><img src="./../img/Function.png"></center>

* T : 함수에 대한 입력 타입이다. 람다식의 매개변수에 어떤 타입이 올지를 정한다.
* R : 함수의 반환 타입이다. 람다식의 반환타입이 어떤 타입일지를 정한다.
```java
Function<Integer, Integer> func = n -> n + n;
Function<Integer, Integer> func2 = n -> n * n;
System.out.println(func.apply(1)); //2
System.out.println(func.apply(2)); //4
System.out.println(func.compose(func2).apply(2)); //8
System.out.println(func.andThen(func2).apply(2)); //16
```
* apply(arg) : apply()에 주어진 파라미터를 람다식에 적용해 반환받는다.
* compose(Fucntion) : Function을 연결하는 함수이다. compose에 파라미터로 주어진 Function을 적용한 후 다음 함수를 실행한다. func.compose(func2).apply(2)에서 func2를 실행한 후 func을 실행하기 때문에 결과는 (2*2)+(2*2)=8이 된다.
* andThen(Function) : compose와 반대이다. 함수를 실행한 후 주어진 Function을 실행한다. func.andThen(func2).apply(2)에서 func을 실행한 후 func2를 실행하기 때문에 결과는 (2+2)*(2+2)=16이 된다.
## Predicate
<center><img src="./../img/Predicate.png"></center>

* T : 함수에 대한 입력 타입이다. 람다식의 매개변수에 어떤 타입이 올지를 정한다. 반환타입은 boolean이다.
```java
Predicate<Integer> func = i -> i > 10;
Predicate<Integer> func2 = i -> i < 20;
Predicate<Integer> negateFunc = func.negate();
Predicate<Integer> isEqualFunc = func.isEqual("abc");
System.out.println(func.test(100)); //true
System.out.println(func.and(func2).test(100)); //false
System.out.println(func.or(func2).test(100)); //true
System.out.println(negateFunc.test(5)); //true
System.out.println(isEqualFunc.test("abc")); //true
```
* test(arg) : test()에 주어진 파라미터를 람다식에 적용해 반환받는다.
* and(Predicate) : Predicate를 and 연산으로 연결하는 함수이다.
* or(Predicate) : Predicate를 or 연산으로 연결하는 함수이다.
* negate() : Predicate가 리턴하는 값의 반대되는 값을 리턴하는 Predicate를 리턴한다. Predicate에 not 연산이 붙었다고 보면된다.
* isEqual(arg) : isEqual()의 파라미터로 전달된 객체와 같은지를 비교하는 Predicate를 리턴한다.
* stream의 filter()의 파라미터로 Predicate를 넣을 수 있다.
## Consumer
<center><img src="./../img/consumer.png"></center>

* T : 함수에 대한 입력 타입이다. 람다식의 매개변수에 어떤 타입이 올지를 정한다. 반환타입은 void이다.
```java
Consumer<List<Integer>> func = nums -> {
    for(int i = 0; i < nums.size(); i++){
        nums.set(i, nums.get(i) * nums.get(i));
    }
}
Consumer<List<Integer>> func2 = nums -> {
    for(num : nums){
        System.out.println(num);
    }
}
List<Integer> nums = new ArrayList<>({0, 1, 2, 3, 4})
System.out.println(func2.accept(nums)); //{0, 1, 2, 3, 4}
System.out.println(func.andThen(func2).accept(nums)) //{0, 1, 4, 9, 16}
```
* accept(arg) : accpet()에 주어진 파라미터를 람다식에 적용해 반환받는다..
* andThen(Consumer) : Consumer를 연결하는 함수이다. 앞의 Consumer부터 실행된 후 뒤에 적용된 Consumer가 실행된다.
## Supplier
<center><img src="./../img/Supplier.png"></center>

* T : 함수의 반환 타입이다. 람다식의 반환타입이 어떤 타입일지를 정한다.
```java
Supplier<String> func = () -> "hello world";
System.out.println(func.get()); //hello world
```
* get() : Supplier의 람다식 리턴값을 반환받는다.
## UnaryOperator
<center><img src="./../img/UnaryOperator.png"></center>

* T : 함수의 입력 타입, 반환 타입이다. 람다식의 매개변수 타입과 반환타입이 같다.
* Fucntion을 상속 받는 인터페이스다.
```java
UnaryOperator<Integer> func = n -> n + n;
UnaryOperator<Integer> func2 = n -> n * n;
System.out.println(func.apply(3)); //6
System.out.println(func.compose(func2).apply(2)); //8
System.out.println(func.andThen(func2).apply(2)); //16
```
* Function의 함수와 같다.