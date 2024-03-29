# Go 기본
## main.go
```go
package main

import "fmt"

func main() {
	fmt.Println("Hello World")
}
```
* go 파일을 컴파일 후 실행하기 위해서는 패키지 이름이 main, 함수 이름이 main인 main.go 파일을 생성해야한다.
* go 는 마지막에 세미콜론(;)을 찍어줄 필요 없다.
* 같은 디렉토리에 여러 패키지가 있을 수 없다. 그리고 main 패키지에는 main.go 파일만 있을 수 있다.
## function
### public, default
```go
package hello

import "fmt"

func helloWorld() {
	fmt.Println("Hello World")
}

func HelloWorld() {
	fmt.Println("Hello World")
}
```
```go
package main

import "study/hello"

func main() {
	hello.HelloWorld()
	//hello.helloWorld() 호출 불가
}
```
* 함수 이름이 소문자로 시작하면 default와 같다. 같은 패키지 내에서만 호출할 수 있다.
* 함수 이름이 대문자로 시작하면 public과 같다. 다른 패키지에서도 호출이 가능하다.
* 만약 함수내에 변수나 상수를 정의해놓고 사용하지 않으면 에러를 발생시킨다.
### 파라미터
```go
package main

import "fmt"

func multiple1(a int, b int) int {
	return a * b
}

func multiple2(a, b int) int {
	return a * b
}

func main() {
	fmt.Println(multiple1(3, 3))
	fmt.Println(multiple2(2, 3))
}
```
* 함수의 리턴타입은 파라미터 다음에 명시해줘야한다. 만약 리턴타입을 명시하지 않으면 void로 실행된다.
* 파라미터의 타입도 파라미터 뒤쪽에 명시해줘야 한다. 같은 타입 여러개가 연속적으로 있다면 가장 마지막에 한번만 명시해줘도 된다.(multipel1과 multiple2는 같은 함수이다.)
### 반환값
```go
package main

import (
	"fmt"
	"strings"
)

// func lenAndUpper(name string) (int, string) {
// 	return len(name), strings.ToUpper(name)
// }

//위의 함수와 같은 동작을 함
func lenAndUpper(name string) (length int, uppercase string) {
	
	// length와 uppercase는 이미 생성되어 있기 때문에 ':='를 쓰면 안된다.
	// length := len(name)
	// uppercase := strings.ToUpper(name)

	length = len(name)
	upperCase = strings.ToUpper(name)
	return
}

func main() {
	nameLength1, uppercaseName := lenAndUpper("lee")
	nameLength2, _ := lenAndUpper("lee")
    //nameLength := lenAndUpper("lee") -> 에러
	fmt.Println(nameLength1, uppercaseName)
	fmt.Println(nameLength2)
}
```
* go에서는 함수에서 여러개의 값을 리턴해 줄 수 있다.
* 반환 타입을 명시할 때 괄호안에 여러개를 넣어서 명시해주면 된다.
* 만약 반환되는 리턴값 중 특정 값만 사용하고 싶다면 사용하지 않는 리턴 값은 `_`를 써서 무시해 줄 수 있다. 만약 `_`를 쓰지 않고 원하는 값만 받으려 하면 에러가 난다.
* naked return : 함수의 반환값을 반환타입에 명시할 수 있다. return 시에 어떤 값을 리턴할지 명시할 필요가 없다.(리턴값을 명시하기를 원한다면 명시해도 돌아간다.)
### ...(정해지지 않은 개수의 파라미터)
```go
package main

import "fmt"

func printAll(s ...string) {
	fmt.Println(s)
}

func main() {
	printAll("owefij", "woefij", "eriv", "leribu", "oefij")
}
```
* 만약 정해지지 않은 개수의 같은 타입의 파라미터를 받으려면 java 처럼 `...`을 사용하면 된다. 단, `...`을 타입 앞에 붙여준다.
### defer
```go
package main

import "fmt"

func multiple1(a int, b int) int {
	defer fmt.Println("done")
	return a * b
}

func multiple2(a, b int) int {
	defer fmt.Println("done")
	return a * b
}

func main() {
	fmt.Println(multiple1(3, 3))
	fmt.Println(multiple2(2, 3))
}
```
* 함수가 끝나고 나서 특정 코드를 실행시키고 싶을 때 defer를 사용하면 된다.
## 변수, 상수
```go
package main

import "fmt"

//s4 := "문자열" -> 사용 불가

var(
	s4 = "문자열"
	s5 = "문자열"
)

func main() {
	const s1 string = "문자열"
    var s2 string = "문자열"
    s3 := "문자열" // 축약형, var로 생성
    //s1 = "문자열바꾸기" -> 에러
    s2 = "문자열바꾸기"
	fmt.Println(s1)
	fmt.Println(s2)
	fmt.Println(s3)
	fmt.Println(s4)
	fmt.Println(s5)
}
```
* const는 상수, var은 변수로 생성된다.
* 축약형은 함수 내에서만 사용가능하다.
* 축약형을 사용하면 var로 생성되고 타입은 처음 변수를 초기화할 때 정해진다.
* 괄호를 사용하면 상수나 변수를 한번에 여러개 생성이 가능하다.
## 반복문
```go
package main

import "fmt"

func addAll(numbers ...int) int {
	sum := 0

	// for i := 0; i < len(numbers); i++ {
	// 	sum += numbers[i]
	// }

	// for index, number := range numbers {
	// 	sum += number
	// 	fmt.Println(index)
	// }

	for _, number := range numbers {
		sum += number
	}
	return sum
}

func main() {
	sum := addAll(1, 2, 3, 4, 6, 8)
	fmt.Println(sum)	
}
```
* range를 사용하는 방법과 C언어 처럼 사용하는 방법 2가지가 있다.
* range를 사용할 때는 첫번째 변수에는 인덱스가 저장되고 두번째 변수에는 값이 저장된다. 만약 인덱스를 무시하고 싶다면 `_`를 써줘야한다.
## 조건문
### if, switch
```go
package main

import "fmt"

func pointIf(score int) {

	// if score >= 90 {
	// 	fmt.Println("good")
	// 	return
	// }
	// fmt.Println("bad")

	if strict := score - 5; strict >= 90 {
		fmt.Println("good")
		return
	}
	fmt.Println("bad")
}

func pointSwitch(score int) {

	// switch {
	// case score > 90:
	// 	fmt.Println("good")
	// 	return
	// default:
	// 	fmt.Println("bad")
	// 	return
	// }

	// switch score {
	// case 100:
	// 	fmt.Println("good")
	// 	return
	// default:
	// 	fmt.Println("bad")
	// 	return
	// }
	
	switch strict := score - 5; {
	case strict > 90:
		fmt.Println("good")
		return
	default:
		fmt.Println("bad")
		return
	}
}

func main() {
	pointIf(94) // bad
	pointIf(95) // good
	pointSwitch(94) // bad
	pointSwitch(95) // good
}
```
* go에서는 C에서나 자바처럼 괄호를 쓸 필요가 없다.
* 조건문 안에서만 사용할 변수를 정의할 수 있다.
## 포인터
* C언어랑 같음
## Struct
```go
package banking

type account struct{
	owner string
	balance int
}

func NewAccount(owner string) *account{
	return &account{owner: owner, balance: 0}
}

// account의 복사본을 받아옴(main에서의 account와 다른 구조체이기 때문에 값이 바뀌지 않음)
// func (a account) Deposit(money int){
// 	a.balance += money
// }

func (a *account) Deposit(money int){
	a.balance += money
}
```
```go
package main

import (
	"fmt"
	"study/banking"
)

func main() {
	account := banking.NewAccount("Lee")
	account.Deposit(10)
	fmt.Println(*account)
}
```
* C언어의 구조체와 같다.
* Go에는 클래스가 없는데 구조체를 사용하여 클래스처럼 사용할 수 있다. 구조체의 이름과 멤버변수의 맨 앞글자를 소문자로 해서 default으로 정의하고 같은 패키지에서 해당 구조체를 생성하는 함수(생성자)와 함수를 정의해서 클래스 처럼 사용한다.
* 생성자를 만들 때 반환값을 포인터로 넘기지 않는다면 반환값을 할당할 때 해당 구조체 자체를 넘기는게 아니라 구조체의 값만을 복사하기 때문에 새로운 구조체가 생성된다. 그러길 원하지 않는다면 포인터로 넘기는게 좋다.(포인터로 넘기지 않는다면 `account := banking.NewAccount("Lee", 100000)`에서 `banking.NewAccount("Lee", 100000)`와 그것을 할당받은 `account`는 같은 값을 가지지만 다른 주소를 가진다.)
* 함수의 리시버를 포인터로 하면 함수에서의 구조체와 메인에서의 구조체는 같은 구조체이다. 그렇기 때문에 함수에서의 변경사항이 메인에서도 반영이 된다. 하지만 리시버를 포인터로 하지 않으면 함수에서의 구조체는 메인에서의 구조체의 복사본이기 때문에 함수에서 값을 변경한다 할지라도 메인에서 반영되지 않는다.
## 에러
```go
package banking

import "errors"

type account struct {
	owner   string
	balance int
}

func NewAccount(owner string) *account {
	return &account{owner: owner, balance: 10}
}

func (a *account) Withdraw(money int) error {
	if a.balance < money {
		return errors.New("잔액이 부족합니다.")
	}
	a.balance -= money
	return nil // null과 같음
}
```
```go
package main

import (
	"fmt"
	"log"
	"study/banking"
)

func main() {
	account := banking.NewAccount("Lee")
	error := account.Withdraw(100)
	if(error != nil){
		log.Fatalln(error) //에러가 일어난 시간과 에러메세지를 출력하고 프로그램을 강제로 종료함
	}
}
```
* Go에는 try-catch문이나 exception이 없고 대신에 error가 존재한다.
* Go에서는 직접 에러를 체크하는 로직을 짜야하는데 만약 함수에서 에러를 체크하고 싶다면 에러를 반환해줘야한다. 이렇게 반환된 에러를 체크하는 로직을 짜서 에러를 핸들링할 수 있다.
## toString
```go
package banking

import "fmt"

type account struct {
	owner   string
	balance int
}

func NewAccount(owner string) *account {
	return &account{owner: owner, balance: 10}
}

func (a account) String() string{
	//string으로 포매팅 해주는 함수
	return fmt.Sprint("소유자 : ", a.owner, "\n잔고 : ", a.balance)
}
```
```go
package main

import (
	"fmt"
	"study/banking"
)

func main() {
	account := banking.NewAccount("Lee")
	fmt.Println(account)
}
```
* 자바에서의 toString()처럼 String() 함수를 정의해주면 구조체를 출력할 때 String()에서 반환하는 문자열을 출력해준다.
## type
```go
package mapping

import "errors"

type Map map[string]string

func (m Map) Search(s string) (string, error) {
	value, exist := m[s]
	if exist {
		return value, nil
	}
	return "", errors.New("찾을 수 없는 문자열입니다.")
}
```
```go
package main

import (
	"fmt"
	"study/mapping"
)

func main() {
	m := mapping.Map{}
	m["test"] = "test"
	fmt.Println(m)
	test, err := m.Search("test")
	if err == nil {
		fmt.Println(test)
	}else {
		fmt.Println(err)
	}
}
```
* Go 에서는 type 키워드를 통해 alias를 만들 수 있다.
* struct와 마찬가지로 함수를 만들 수도 있다.
## GoRoutine, Channel
```go
package main

import (
	"fmt"
	"time"
)

func main() {
	//Channel
	channel := make(chan bool)
	//GoRoutine
	go goRoutineTest(9, channel)
	go goRoutineTest(10, channel)
	fmt.Println(<-channel)
	fmt.Println(<-channel)
}

// func goRoutineTest(num int, channel chan bool) { // channel에서 데이터 보내기 받기 둘다 가능
func goRoutineTest(num int, channel chan<- bool) { // channel에 데이터를 보내기만 가능
	time.Sleep(2)
	if num >= 10 {
		channel <-true
	}else {
		channel <-false
	}
}
```
* GoRoutine은 Go에서 사용하는 경량 쓰레드이다. 일반적인 쓰레드와의 차이점은 사용자 쓰레드나 커널 쓰레드는 쓰레드를 생성하고 삭제를 할 때 system call을 통해 OS에 요청을 해야한다. 따라서 오버헤드가 많이 발생할 수 있는데 GoRoutine은 이러한 system call 없이 유저 스페이스에서 동작을 완료하기 때문에 오버헤드가 적다.
* GoRoutine은 값을 다른 함수에 전달하기 위해 Channel을 사용한다.
* 위의 예시에서 `fmt.Println(<-channel)`가 없다면 메인 함수는 GoRoutine이 동작을 완료할 때까지 기다리지 않는다. 하지만 메인함수가 `fmt.Println(<-channel)`를 만나면 메인함수는 Channel에서 받을 값이 있다는 것을 알고 Channel에서 값을 받을 때 까지 기다리게 된다. 만약 Channel에서 받을 값보다 더 많은 `<-channel`이 있다면 deadlock이 걸리면서 에러가 뜬다.
* 함수의 파라미터에 들어있는 채널의 `chan`키워드를 그대로 사용하면 해당 함수에서는 채널에 값을 보내기, 받기가 둘 다 가능하고, `chan<-`을 사용하면 보내기만 가능하고, `<-chan`을 사용하면 받기만 가능하다.