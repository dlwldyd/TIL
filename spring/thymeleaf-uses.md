 # 타입리프 기본 사용법
```java
@Getter
@Setter
@ToString
@AllArgsConstructor
public class User {
    private String username;
    private int age;
}
```
* 이 문서에서 사용하는 User 객체
## text, \[[...]]
```html
<ul>
    <li>th:text 사용 <span th:text="${data}"></span></li>
    <li>컨텐츠 안에서 직접 출력하기 = [[${data}]]</li>
</ul>
```
* HTML의 콘텐츠에 데이터를 출력할 때는 th:text 를 사용하면 된다.
* HTML 테그의 속성이 아니라 HTML 콘텐츠 영역안에서 직접 데이터를 출력하고 싶으면 [[...]] 를 사용하면 된다. ex)  [[${data}]]
## utext, \[(...)]
```html
<ul>
    <li>th:utext = <span th:utext="${data}"></span></li>
    <li><span th:inline="none">[(...)] = </span>[(${data})]</li>
</ul>
```
* th:inline="none" : 타임리프는 [[...]] 를 해석하기 때문에, 화면에 [[...]] 글자를 보여줄 수 없다. 이 테그 안에서는 타임리프가 해석하지 말라는 옵션이다.
* 이스케이프 기능을 사용하지 않으려면 text가 아닌 utext를 사용해야 한다.
* 이스케이프 기능을 사용하지 않고 HTML 콘텐츠 영역안에서 직접 데이터를 출력하고 싶으면 [(...)]를 사용하면 된다.
## 변수 표현식
```html
<ul>Object
    <li>${user.username} = <span th:text="${user.username}"></span></li>
    <li>${user['username']} = <span th:text="${user['username']}"></span></li>
    <li>${user.getUsername()} = <span th:text="${user.getUsername()}"></span></
li>

<ul>List
    <li>${users[0].username} = <span th:text="${users[0].username}"></span></li>
    <li>${users[0]['username']} = <span th:text="${users[0]['username']}"></span></li>
    <li>${users[0].getUsername()} = <span th:text="${users[0].getUsername()}"></span></li>
</ul>

<!--key는 userA 와 userB 이다.-->
<ul>Map
    <li>${userMap['userA'].username} = <span th:text="${userMap['userA'].username}"></span></li>
    <li>${userMap['userA']['username']} = <span th:text="${userMap['userA']['username']}"></span></li>
    <li>${userMap['userA'].getUsername()} = <span th:text="${userMap['userA'].getUsername()}"></span></li>
```
* 객체로 넘겼을 때 user.username, user['username'], user.getUsername()로 접근할 수 있다.
* 리스트로 넘겼을 때 users[index].username, list.get(index).getUsername(), users[index]['username']로 접근할 수 있다.
* 맵으로 넘겼을 때 userMap['key'].username, userMap['key']['username'], userMap['key'].getUsername()로 접근 할 수 있다.
## 지역 변수 선언
```html
<!--users는 List이다.-->
<div th:with="first=${users[0]}">
    <p>처음 사람의 이름은 <span th:text="${first.username}"></span></p>
</div>
```
* th:with 를 사용하면 지역 변수를 선언해서 사용할 수 있다.
## 타임리프 기본 객체
```html
<!--식 기본 객체 (Expression Basic Objects)-->
<ul>
    <li>request = <span th:text="${#request}"></span></li>
    <li>response = <span th:text="${#response}"></span></li>
    <li>session = <span th:text="${#session}"></span></li>
    <li>servletContext = <span th:text="${#servletContext}"></span></li>
    <li>locale = <span th:text="${#locale}"></span></li>
</ul>
<!--편의 객체-->
<ul>
    <li>Request Parameter = <span th:text="${param.paramData}"></span></li>
    <li>session = <span th:text="${session.sessionData}"></span></li>
    <!--hello함수는 매개변수를 그대로 리턴하는 함수-->
    <li>spring bean = <span th:text="${@helloBean.hello('Spring!')}"></span></li>
</ul>
```
* 타임리프는 다음과 같은 기본 객체들을 제공한다.
  + ${#request}
  + ${#response}
  + ${#session}
  + ${#servletContext}
  + ${#locale}
* 다음과 같은 편의 객체도 제공한다.
  + HTTP 요청 파라미터 접근: param
    - ex) ${param.paramData}
  + HTTP 세션 접근: session
    - ex) ${session.sessionData}
  + 스프링 빈 접근: @
    - ex) ${@helloBean.hello('Spring!')}
## JAVA8 날짜용 유틸리티 객체
```html
<span th:text="${#temporals.format(localDateTime, 'yyyy-MM-dd HH:mm:ss')}"></
span>
```
* java8에서 사용 가능
* format외에도 다양한 출력 형식이 있다.
## URL 링크
```html
<ul>
    <li><a th:href="@{/hello}">basic url</a></li>
    <li><a th:href="@{/hello(param1=${param1}, param2=${param2})}">hello query param</a></li>
    <li><a th:href="@{/hello/{param1}/{param2}(param1=${param1}, param2=${param2})}">path variable</a></li>
    <li><a th:href="@{/hello/{param1}(param1=${param1}, param2=${param2})}">path variable + query parameter</a></li>
</ul>
```
* 타임리프에서 URL을 생성할 때는 @{...} 문법을 사용하면 된다.
* @{/hello(param1=${param1}, param2=${param2})} -> /hello?param1=data1&param2=data2
* @{/hello/{param1}/{param2}(param1=${param1}, param2=${param2})} -> /hello/data1/data2
* @{/hello/{param1}(param1=${param1}, param2=${param2})} -> /hello/data1?param2=data2
## 리터럴
```html
<ul>
    <li>'hello' + ' world!' = <span th:text="'hello' + ' world!'"></span></li>
    <li>'hello world!' = <span th:text="'hello world!'"></span></li>
    <li>'hello ' + ${data} = <span th:text="'hello ' + ${data}"></span></li>
    <li>리터럴 대체 |hello ${data}| = <span th:text="|hello ${data}|"></span></li>
</ul>
```
* 타임리프에서 문자 리터럴은 항상 작은 따옴표로 감싸야 한다.
* 문자가 공백 없이 쭉 이어진다면 하나의 의미있는 토큰으로 인지해서 작은 따옴표를 생략할 수 있다. ex) \<span th:text="hello">
* 리터럴 대체 문법을 사용하면 변수를 사용할 때 +연산자를 이용할 필요가 없다.
## 비교 연산
```html
<ul>
    <li>1 > 10 = <span th:text="1 &gt; 10"></span></li>
    <li>1 gt 10 = <span th:text="1 gt 10"></span></li>
    <li>1 >= 10 = <span th:text="1 >= 10"></span></li>
    <li>1 ge 10 = <span th:text="1 ge 10"></span></li>
    <li>1 == 10 = <span th:text="1 == 10"></span></li>
    <li>1 != 10 = <span th:text="1 != 10"></span></li>
</ul>
```
* HTML 엔티티를 사용해야 하는 부분을 주의하자
## elvis 연산자
```html
<ul>
    <li>${data}?: '데이터가 없습니다.' = <span th:text="${data}?: '데이터가 없습니다.'"></span></li>
    <li>${nullData}?: '데이터가 없습니다.' = <span th:text="${nullData}?: '데이터가 없습니다.'"></span></li>
</ul>
```
* 조건식의 편의 버전이다.
* true면 ?:의 앞부분이 출력되고 false면 뒷부분이 출력된다.
## No-Operation
```html
<ul>
    <li>${data}?: _ = <span th:text="${data}?: _">데이터가 없습니다.</span></li>
    <li>${nullData}?: _ = <span th:text="${nullData}?: _">데이터가 없습니다.</span></li>
</ul>
```
* : _ 인 경우 마치 타임리프가 실행되지 않는 것 처럼 동작한다.(해당 태그 자체가 출력 되지 않음)
## 타임리프 태그 속성
```html
<!--속성 설정-->
<input type="text" name="mock" th:name="userA" />
```
```html
<!--속성 추가-->
- th:attrappend = <input type="text" class="text" th:attrappend="class='large'" /><br/>
- th:attrprepend = <input type="text" class="text" th:attrprepend="class='large'" /><br/>
- th:classappend = <input type="text" class="text" th:classappend="large" /><br/>
```
```html
<!--checked 처리-->
- checked o <input type="checkbox" name="active" th:checked="true" /><br/>
- checked x <input type="checkbox" name="active" th:checked="false" /><br/>
- checked=false <input type="checkbox" name="active" checked="false" /><br/>
```
* 타임리프로 속성을 지정하면 타임리프는 기존 속성을 타임리프로 지정한 속성으로 대체한다. 기존 속성이 없으면 새로 만든다.
* th:attrappend : 속성 값의 뒤에 값을 추가한다.(text -> textlarge)
* th:attrprepend : 속성 값의 앞에 값을 추가한다.(text -> largetext)
* th:classappend : class 속성에 한칸 띄우고 추가한다.(text -> text large)
* 기존 HTML에서는 \<input type="checkbox" name="active" checked="false" /> 의 경우에도 checked 속성이 있기 때문에 checked 처리가 되어버린다. 그러나 타임리프의 th:checked="false"를 사용하면 checked 속성 자체를 제거한다.(\<input type="checkbox" name="active" th:checked="false" /> -> 타임리프 렌더링 후: <input type="checkbox" name="active" />)
## 반복
```html
<table border="1">
    <tr>
        <th>count</th>
        <th>username</th>
        <th>age</th>
        <th>etc</th>
    </tr>
    <!--<tr th:each="user, userStat : ${users}">-->
    <tr th:each="user : ${users}">
        <td th:text="${userStat.count}">username</td>
        <td th:text="${user.username}">username</td>
        <td th:text="${user.age}">0</td>
        <td>
            index = <span th:text="${userStat.index}"></span>
            count = <span th:text="${userStat.count}"></span>
            size = <span th:text="${userStat.size}"></span>
            even? = <span th:text="${userStat.even}"></span>
            odd? = <span th:text="${userStat.odd}"></span>
            first? = <span th:text="${userStat.first}"></span>
            last? = <span th:text="${userStat.last}"></span>
            current = <span th:text="${userStat.current}"></span>
        </td>
    </tr>
</table>
```
* 타임리프에서 반복은 th:each 를 사용한다. 추가로 반복에서 사용할 수 있는 여러 상태 값을 지원한다.
* th:each는 List 뿐만 아니라 배열, java.util.Iterable , java.util.Enumeration 을 구현한 모든 객체를 반복에 사용할 수 있다. Map 도 사용할 수 있는데 이 경우 변수에 담기는 값은 Map.Entry다.
* 반복 상태 유지 기능
  + index : 0부터 시작하는 값
  + count : 1부터 시작하는 값
  + size : 전체 사이즈
  + even , odd : 홀수, 짝수 여부( boolean )
  + first , last :처음, 마지막 여부( boolean )
  + current : 현재 객체
## 조건식
```html
<!--if, unless-->
<table border="1">
    <tr>
        <th>count</th>
        <th>username</th>
        <th>age</th>
    </tr>
    <tr th:each="user, userStat : ${users}">
        <td th:text="${userStat.count}">1</td>
        <td th:text="${user.username}">username</td>
        <td>
            <span th:text="${user.age}">0</span>
            <span th:text="'미성년자'" th:if="${user.age lt 20}"></span>
            <!--<span th:text="'미성년자'" th:unless="${user.age ge 20}"></span>-->
        </td>
    </tr>
</table>
```
```html
<!--switch-->
<table border="1">
    <tr>
        <th>count</th>
        <th>username</th>
        <th>age</th>
    </tr>
    <tr th:each="user, userStat : ${users}">
        <td th:text="${userStat.count}">1</td>
        <td th:text="${user.username}">username</td>
        <td th:switch="${user.age}">
            <span th:case="10">10살</span>
            <span th:case="20">20살</span>
            <span th:case="*">기타</span>
        </td>
    </tr>
</table>
```
* unless는 if의 반대이다.
* 타임리프는 해당 조건이 맞지 않으면 태그 자체를 렌더링하지 않는다.
* switch에서 * 는 만족하는 조건이 없을 때 사용하는 디폴트이다.
## safe navigation operation
```html
<div th:if="${errors?.containsKey('globalError')}" class="field-error" th:text="${errors['globalError']}">전체 오류 메시지</div>
```
* errors?.containsKey(itemName)에서
errors가 null일때 NullPointerExeception이 발생하는 대신 null값을 반환한다. 만약 null이 아니라면 errors.containsKey(itemName)함수가 실행된다.
* th:if에서 null은 실패로 처리되므로 렌더링 될 때 해당 태그는 출력되지 않는다.
## 주석
```html
<h1>1. 표준 HTML 주석</h1>
<!--
<span th:text="${data}">html data</span>
-->

<h1>2. 타임리프 파서 주석</h1>
<!--/* [[${data}]] */-->

<h1>3. 타임리프 프로토타입 주석</h1>
<!--/*/
<span th:text="${data}">html data</span>
/*/-->
```
* 자바스크립트의 표준 HTML 주석은 타임리프가 렌더링 하지 않고, 그대로 남겨둔다.
* 타임리프 파서 주석은 타임리프가 렌더링에서 주석 부분을 제거한다.
* 타임리프 프로토타입 주석은 HTML 파일을 그대로 열어보면 주석처리가 되지만, 타임리프를 렌더링 한 경우 렌더링 되어 문서가 보여지게 된다.
## 블록
```html
<th:block th:each="user : ${users}">
    <div>
        사용자 이름1 <span th:text="${user.username}"></span>
        사용자 나이1 <span th:text="${user.age}"></span>
    </div>
    <div>
        요약 <span th:text="${user.username} + ' / ' + ${user.age}"></span>
    </div>
</th:block>
```
* <th:block>는 렌더링시 제거된다.
* 여러 태그를 함께 반복문을 돌려야 하는 등 특수한 경우에 사용하는 태그이다.
## 자바스크립트 인라인
```html
<!-- 자바스크립트 인라인 사용 전 -->
<script>
 var username = [[${user.username}]];
 var age = [[${user.age}]];
 //자바스크립트 내추럴 템플릿
 var username2 = /*[[${user.username}]]*/ "test username";
 //객체
 var user = [[${user}]];
</script>

<!-- 자바스크립트 인라인 사용 후 -->
<script th:inline="javascript">
    var username = [[${user.username}]];
    var age = [[${user.age}]];
    //자바스크립트 내추럴 템플릿
    var username2 = /*[[${user.username}]]*/ "test username";
    //객체
    var user = [[${user}]];
</script>
```
```html
<!--자바스크립트 인라인 사용 전 결과-->
<script>
    var username = userA;// 오류 발생
    var age = 10;
    //userA는 주석처리 된다.
    var username2 = /*userA*/ "test username";
    //객체의 toString()이 호출된 값이 담긴다.
    var user = BasicController.User(username=userA, age=10);
</script>
```
```html
<!--자바스크립트 인라인 사용 후 결과-->
<script>
    //자동으로 쌍따옴표가 붙는다.
    var username = "userA";
    var age = 10;
    //주석 내의 값이 할당된다.
    var username2 = "userA";
    //객체가 JSON으로 변환되어 담긴다.
    var user = {"username":"userA","age":10};
</script>
```
* 타임리프의 자바스크립트 인라인 기능을 사용하면 쌍따옴표를 신경쓸 필요 없이 문자 타입인 경우 " 를 자동으로 포함해준다. 추가로 자바스크립트에서 문제가 될 수 있는 문자가 포함되어 있으면 이스케이프 처리도 해준다.
* 타임리프는 HTML 파일을 직접 열어도 동작하는 내추럴 템플릿 기능을 제공한다. 자바스크립트 인라인 기능을 사용하면 주석을 활용해서 이 기능을 사용할 수 있다.
* var username2 = /\*[[${user.username}]]*/ "test username";
  + html 파일을 직접 열었을 때
    - 인라인 사용 전 : var username2 = "test username";
    - 인라인 사용 후 : var username2 = "test username";
  + 렌더링 했을 때
    - 인라인 사용 전 : var username2 = "test username";
    - 인라인 사용 후 : var username2 = "userA";
* 타임리프의 자바스크립트 인라인 기능을 사용하면 객체를 JSON으로 자동으로 변환해준다.
## 자바스크립트 인라인 each
```html
<script th:inline="javascript">
    [# th:each="user : ${users}"]
    var user[[${userStat.count}]] = [[${user}]];
    [/]
</script>
```
```html
<!--결과-->
<script>
    var user1 = {"username":"userA","age":10};
    var user2 = {"username":"userB","age":20};
    var user3 = {"username":"userC","age":30};
</script>
```
## 템플릿 조각
```html
<!--templates/template/fragment/footer-->
<!DOCTYPE html>
<html xmlns:th="http://www.thymeleaf.org">
<body>
<footer th:fragment="copy">
    푸터 자리 입니다.
</footer>
<footer th:fragment="copyParam (param1, param2)">
    <p>파라미터 자리 입니다.</p>
    <p th:text="${param1}"></p>
    <p th:text="${param2}"></p>
</footer>
</body>
</html>
```
```html
<!DOCTYPE html>
<html xmlns:th="http://www.thymeleaf.org">
<head>
    <meta charset="UTF-8">
    <title>Title</title>
</head>
<body>
<h1>부분 포함</h1>
    <h2>부분 포함 insert</h2>
    <div th:insert="~{template/fragment/footer :: copy}"></div>
    <h2>부분 포함 replace</h2>
    <div th:replace="~{template/fragment/footer :: copy}"></div>
    <h2>부분 포함 단순 표현식</h2>
    <div th:replace="template/fragment/footer :: copy"></div>
<h1>파라미터 사용</h1>
    <div th:replace="~{template/fragment/footer :: copyParam ('데이터1', '데이터2')}"></div>
</body>
</html>
```
* 부분 포함 insert는 현재 태그 내부에 추가한다.
```html
<h2>부분 포함 insert</h2>
<div>
<footer>
    푸터 자리 입니다.
</footer>
</div>
```
* 부분 포함 replace는 현재 태그를 대체한다.
```html
<h2>부분 포함 단순 표현식</h2>
<footer>
    푸터 자리 입니다.
</footer>
```
* 파라미터를 전달해 동적으로 렌더링 할 수 있다.
```html
<h1>파라미터 사용</h1>
<footer>
    <p>파라미터 자리 입니다.</p>
    <p>데이터1</p>
    <p>데이터2</p>
</footer>
```
## 템플릿 레이아웃
```html
<!--templates/template/layoutEXtend/layoutFile-->
<!DOCTYPE html>
<html th:fragment="layout (title, content)" xmlns:th="http://www.thymeleaf.org">
<head>
    <title th:replace="${title}">레이아웃 타이틀</title>
</head>
<body>
<h1>레이아웃 H1</h1>
<div th:replace="${content}">
    <p>레이아웃 컨텐츠</p>
</div>
<footer>
    레이아웃 푸터
</footer>
</body>
</html>
```
```html
<!DOCTYPE html>
<html th:replace="~{template/layoutExtend/layoutFile :: layout(~{::title}, ~{::section})}" xmlns:th="http://www.thymeleaf.org">
<head>
    <title>메인 페이지 타이틀</title>
</head>
<body>
<section>
    <p>메인 페이지 컨텐츠 1</p>
    <div>메인 페이지 포함 내용 1</div>
</section>
<section>
    <p>메인 페이지 컨텐츠 2</p>
    <div>메인 페이지 포함 내용 2</div>
</section>
</body>
</html>
```
```html
<!--결과-->
<!DOCTYPE html>
<html>
<head>
    <title>메인 페이지 타이틀</title>
</head>
<body>
<h1>레이아웃 H1</h1>
<section>
    <p>메인 페이지 컨텐츠 1</p>
    <div>메인 페이지 포함 내용 1</div>
</section>
<section>
    <p>메인 페이지 컨텐츠 2</p>
    <div>메인 페이지 포함 내용 2</div>
</section>
<footer>
    레이아웃 푸터
</footer>
</body>
</html>
```
* 공통 정보들을 한 곳에 모아두고 공통으로 사용하지만, 각 페이지마다 필요한 정보를 더 추가해서 사용하고 싶을 때 사용하는 방식이다.
* ::title 은 현재 페이지의 title 태그들을 전달한다. 
* ::section 는 현재 페이지의 section 태그들을 전달한다.