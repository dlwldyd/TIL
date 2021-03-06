# 패키지
## http
### http.Get()
```go
res, err := http.Get("https://google.com/") // 구글에 get 메서드 보냄
```
* http 통신과 관련된 기능을 하는 패키지이다.
* Get() 함수 안에 들어가는 문자열의 url에는 `https://`이나 `http://`같은 프로토콜 까지 들어가야한다.
* Get() 함수의 반환값은 GET 요청 후 받은 응답정보와 요청 실패 시 에러이다.
* res.Body 사용 시 res.Body는 입출력 스트림이기 때문에 전부 사용하였으면 `res.Body.Close()`를 통해 닫아줘야지 memory leak이 일어나지 않는다. 
## goquery
```go
res, err := http.Get("https://google.com/")
doc, docErr := goquery.NewDocumentFromReader(res.Body) // 응답의 html document를 읽음
defer res.Body.Close() //res.Body는 입출력 스트림이기 때문에 함수가 끝날 때 닫아줘야지 memory leak이 일어나지 않는다.
sel := doc.Find(".cssSelector") //css selector 사용 가능
sel.Each(func(i int, card *goquery.Selection) { // Each()함수를 샤용해 각 요소에 접근 가능
	fmt.Println(card.Find(".selector>span").Text())
})
size := sel.Length() // selector를 통해 가져온 요소의 개수 알 수 있음
```
* Jquery처럼 html 문서를 핸들링하는 패키지이다.
* Jquery와 함수이름이 거의 같다.
## strconv
```go
strconv.Itoa(10) // 숫자 10을 문자열 "10"으로 변환
```
* 문자열을 다른 형식으로 바꾸거나 다른 형식을 문자열로 바꾸는 기능을 하는 패키지이다.
## strings
```go
strings.Trimspace("          테스트 입니다.         ") // 양 옆 공백 없앰, "테스트 입니다."로 변환됨
strings.Fields("컴퓨터     케이크   사과 빵      가방") // 문자열을 문자열 slice로, {"컴퓨터", "케이크", "사과", "빵", "가방"} 으로 변환
strings.Join({"컴퓨터", "케이크", "사과", "빵", "가방"}, ", ") // 문자열 slice를 문자열로, "컴퓨터, 케이크, 사과, 빵, 가방"으로 변환됨
```
* 문자열을 다루는 패키지이다.