# SecurityContext
<center><img src="./../img/SecurityContext.png"></center>

```java
@Configuration
@EnableWebSecurity
public class SecurityConfig extends WebSecurityConfigurerAdapter {
    
    @Override
    protected void configure(HttpSecurity http) throws Exception {
        SecurityContextHolder.setStrategyName(SecurityContextHolder.MODE_THREADLOCAL);
        // SecurityContextHolder.setStrategyName(SecurityContextHolder.MODE_INHERITABLETHREADLOCAL);
        // SecurityContextHolder.setStrategyName(SecurityContextHolder.MODE_GLOBAL);
    }
}
```
```java
@GetMapping("/thread")
public String thread() {

    Authentication auth1 = SecurityContextHolder.getContext().getAuthentication();

    new Thread(() -> {
        Authentication auth2 = SecurityContextHolder.getContext().getAuthentication();
    }); // SecurityContextHolder 의 mode 에 따라 같은 객체일 수도 있고 다른 객체일 수도 있다.

    return "thread";
}
```
* SecurityContext에는 사용자의 정보가 저장되어있는 객체인 Authentication 객체가 들어있다.
* Authentication 객체에는 principal(사용자 정보), credential(password 정보), authorities(권한정보), details(인증 부가 정보), Authenticated(인증 여부)가 들어있다.
* SecurityContext는 SecurityContextHolder안에 ThreadLocal\<SecurityContext>의 형태로 들어있다. 따라서 SecurityContext는 각각의 스레드 마다 다른 SecurityContext가 참조될 수 있다.
* SecurityContextHolder의 모드
  + MODE_THREADLOCAL : 각각의 스레드마다 다른 SecurityContext를 가진다.(default)
  + MODE_INHERITABLETHREADLOCAL : 클라이언트가 스레드 풀로부터 할당받은 스레드와 그 하위 스레드는 전부 같은 SecurityContext를 가진다. 클라이언트끼리는 다른 SecurityContext를 가진다.
  + SecurityContextHolder.MODE_GLOBAL : 모든 스레드가 같은 SecurityContext를 가진다.
* SecurityContext의 생성, 활용
  1. 클라이언트의 요청을 받으면 SecurityContextPersistenceFilter는 해당 요청의 세션에 Securitycontext가 존재할 경우, 해당 SecurityContext를 반환받아 SecurityContextHolder에 저장하고 다음필터로 넘어가서 보안처리를 진행한 후 클라이언트의 요청을 처리한다. 해당 요청의 세션에 Securitycontext가 존재하지 않을 경우, 새로운 SecurityContext를 반환받아 SecurityContextHolder에 저장하고 인증 필터(로그인을 하는 경우(폼인증 방식) UsernamePasswordAuthenticationFilter, 그 외에는 AnonymousAuthenticationFilter)로 넘어간다.
  2. 인증 필터는 인증 처리를 수행하여 만든(인증에 성공했을 경우) Authentication 객체를 SecurityContext에 저장하고 다음 필터로 넘어가서 보안처리를 진행한 후 클라이언트의 요청을 처리한다.
  3. 클라이언트의 요청을 전부 처리하고 응답을 보낼 때 SecurityContextPersistenceFilter는 이렇게 만들어진 SecurityContext를 세션에 저장한다.
  4. SecurityContextPersistenceFilter는 SecurityContextHolder.clearContext()를 통해 SecurityContextHolder에서 SecurityContext를 삭제한다.(ThreadLocal\<SecurityContext>를 remove()함)