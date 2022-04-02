# 자료구조
## Array, Slic
```go
package main

import "fmt"

func main() {
	name1 := [4]string{"홍길동", "안철수", "김수미"} // Array
	name2 := []string{"홍길동", "안철수", "김수미"} // Slice
	
	name1[3] = "안상국"
	// name2[3] = "안상국" -> 불가

	// name1 = append(name1, "강동원", "허경영") -> 불가
	name2 = append(name2, "강동원", "허경영")
	
	fmt.Println(name1)
	fmt.Println(name2)
}
```
* Array는 크기가 고정되어있는 배열, Slice는 크기가 고정되어 있지 않은 배열이다.
* Slice는 append()를 쓸 수 있지만 Array는 append()를 쓸 수 없다.
* append()는 주어진 Slice를 바꾸는 게 아니라 주어진 Slice에 값을 붙인 새로운 Slice를 반환한다. 따라서 주어진 Slice를 바꾸고 싶다면 `=`를 통해 할당해줄 필요가 있다.
## map
```go
package main

import "fmt"

func main() {
    // m := map[string]string{} -> 빈 map이 생성됨
    // m := make(map[string]string) -> 빈 map이 생성됨, 이거랑 위에거 중에서 선호하는거 쓰면됨
    // var m map[string]string -> m은 빈 map이 아니라 nil임, m을 사용하려면 m에 다른 map을 할당해줘야함
	m := map[string]string{"hello":"world", "helloo":"woorld"}
	for key, value := range m {
		fmt.Println(key, value)
	}
    value, exist := m[hello]
    if exist {
        fmt.Println(value)
    }
    delete(m, "hello")
    value = m[hello]
    fmt.Println(value)

    for key, value := range m {
        fmt.Println(key, value)
    }
}
```
* map의 값을 받을 때 2개의 값을 받을 수 있는데 첫 번째 값은 내가 찾으려는 값이고 두 번째 값은 내가 찾는 값이 있는지 없는지를 나타내는 bool값이다.
* delete(map, key) 함수를 통해 map에 있는 값을 삭제할 수 있다. 만약 map에 해당 key가 없으면 아무런 동작도 하지 않는다.
* map을 반복문을 돌 때 key와 value 값을 받는다.