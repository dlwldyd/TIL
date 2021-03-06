# 쿠키, 세션 처리
## 쿠키
```java
//HttpServletResponse response
Cookie cookie = new Cookie("id", String.valueOf(member.getId()));
response.addCookie(cookie);   
```
```java
@GetMapping("/")
 public String homeLogin(
 @CookieValue(name = "id", required = false) Long id,
Model model) {
    ...
}
```
* 쿠키를 생성하고 HttpServletResponse에 쿠키를 담으면 응답의 헤더의 Set-Cookie 필드에 쿠키가 추가된다.
* @CookieValue를 이용하면 편리하게 쿠키를 조회할 수 있다.
* required=false로 설정하면 쿠키가 없는 경우에도 컨트롤러가 실행된다.
## 세션
```java
//HttpServletRequest request
HttpSession session = request.getSession();
session.setAttribute("loginMember", loginMember);
```
```java
HttpSession session = request.getSession(false);
if (session != null) {
    session.invalidate();
}
```
```java
Member loginMember = (Member) session.getAttribute("loginMember");
```
```java
public String login(@SessionAttribute(name = "loginMember", required = false) Member loginMember, Model model) {
    ...
}
```
* request.getSession()은 세션이 있으면 기존 세션을 반환하고 없으면 새로운 세션을 생성해서 반환한다.
* request.getSession(false)는 세션이 있으면 기존 세션을 반환하고 없으면 세션을 생성하지 않고 null을 반환한다.
* session.setAttribute(name, value)를 통해 세션에 데이터를 보관할 수 있다. 하나의 세션에 여러 값을 저장할 수 있다.
* session.invalidate()를 통해 세션을 제거할 수 있다.
* session.getAttribute(name)을 통해 세션에 보관된 데이터를 조회할 수 있다.
* @SessionAttribute를 이용하면 간단하게 세션에 보관된 데이터를 조회할 수 있다.
```properties
#application.properties
server.servlet.session.tracking-modes=cookie
server.servlet.session.timeout=60
```
* 처음 세션을 생성하면 서버가 세션id를 쿠키뿐만 아니라 URL을 통해서도 전달하는데, 서버 입장에서는 클라이언트의 브라우저가 쿠키를 지원하는지 모르므로 세션id를 쿠키로도 전달하고 URL로도 전달한다. 만약 이러한 URL 전달 방식을 사용하지 싫다면 `application.properties`에 `server.servlet.session.tracking-modes=cookie`를 추가하면 된다.
* 서버는 default로 사용자가 마지막으로 서버에 요청한 시점으로 부터 1800초(30분)동안 요청이 없다면 해당 세션을 제거한다. 만약 이러한 timeout 시간을 설정하고 싶다면 `application.properties`에 `server.servlet.session.timeout=(timeout 시간)`을 추가하면 된다.