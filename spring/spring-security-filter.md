# 스프링 시큐리티 설정 API, 스프링 시큐리티 필터
## 스프링 시큐리티 설정
```java
@Configuration
@EnableWebSecurity //스프링 시큐리티 configuration 객체에 해당 어노테이션을 꼭 붙여줘야함
@RequiredArgsConstructor
public class SecurityConfig extends WebSecurityConfigurerAdapter {
    
    @Override
    protected void configure(HttpSecurity http) throws Exception {
        
        http.authorizeRequests()
                .anyRequest().authenticated()

        .and().formLogin();
    }
}
```
* WebSecurityConfigurerAdapter를 상속받은 클래스에 configure(HttpSecurity http)를 오버라이드 하면 form 인증에 대한 설정을 할 수 있다.
* 해당 설정 클래스에는 @EnableWebSecurity를 꼭 붙여주어야 한다.(스프링 시큐리티 설정에 필요한 객체를 스프링 빈으로 등록함)
* 아무런 설정도 하지 않는다면 스프링 시큐리티는 (ID : user, password : 임의의 값)을 기본적으로 제공해준다.
* 스프링 시큐리티가 기본으로 제공해주는 아이디와 패스워드를 바꾸고 싶다면 application.properties에서 `spring.security.user.name`으로 아이디를 바꿀 수 있고, `spring.security.user.name`으로 패스워드를 바꿀 수 있다.
## UsernamePasswordAuthenticationFilter
<center><img src="./../img/UsernamePasswordAuthenticationFilter.png"></center>

```java
//인가 정책
http.authorizeRequests() //http request 인가 설정 시작
    .anyRequest().authenticated() //모든 request 에 대해 (인가에 대한)인증을 수행함

    //인증 정책
    .and()
    .formLogin() //form 로그인 방식으로 인증 수행, 설정 시작
    .loginPage("/loginPage") //로그인 페이지 지정(default : /login)
    // .defaultSuccessUrl("/") //인증이 성공했을 때 이동할 url
    // .failureUrl("/loginPage") // 인증이 실패했을 때 이동할 url
    .usernameParameter("userId") //아이디가 어떤 파라미터명으로 들어올지 지정(default : username)
    .passwordParameter("passwd") //패스워드가 어떤 파라미터명으로 들어올지 지정(default : password)
    .loginProcessingUrl("/login_proc") //로그인 폼 데이터가 어떤 url 로 올지 지정(form 태그의 action 속성, default : /login)
    .successHandler(new AuthenticationSuccessHandler() { //AuthenticationSuccessHandler 는 인터페이스
        //인증에 성공했을 때 실행할 핸들러를 successHandler() 에 넣어준다.(람다식 가능)
        /*
        핸들러는 하나만 넣을 수 있다. 따라서 defaultSuccessUrl()은 해당 url 로 redirect 하는 핸들러를
        추가하는 메서드이기 때문에 내가 따로 핸들러를 만들어서 넣는다면 해당 핸들러에 redirect 로직을 넣어야 한다.
        */
        
        @Override
        public void onAuthenticationSuccess(HttpServletRequest request, HttpServletResponse response, Authentication authentication) throws IOException, ServletException {
            //인증 성공 후 실행할 로직을 넣어준다.
            //authentication 에는 인증에 성공한 사용자의 정보와 사용자의 권한 정보가 들어있다.

            System.out.println("request.getRequestURL() = " + request.getRequestURL());
            Enumeration<String> headerNames = request.getHeaderNames();
            while (headerNames.hasMoreElements()) {
                String name = headerNames.nextElement();
                String header = request.getHeader(name);
                System.out.println(name + " = " + header);
            }
            System.out.println("authentication.getName() = " + authentication.getName());

            response.sendRedirect("/");
        }
    })
    .failureHandler(new AuthenticationFailureHandler() { //AuthenticationFailureHandler 는 인터페이스
        //인증에 성공했을 때 실행할 핸들러를 failureHandler() 에 넣어준다.(람다식 가능)
        /*
        핸들러는 하나만 넣을 수 있다. 따라서 failureUrl()은 해당 url 로 redirect 하는 핸들러를
        추가하는 메서드이기 때문에 내가 따로 핸들러를 만들어서 넣는다면 해당 핸들러에 redirect 로직을 넣어야 한다.
        */

        @Override
        public void onAuthenticationFailure(HttpServletRequest request, HttpServletResponse response, AuthenticationException exception) throws IOException, ServletException {
            //인증 성공 후 실행할 로직을 넣어준다.

            //예외 출력
            System.out.println("exception.getMessage() = " + exception.getMessage());

            response.sendRedirect("/loginPage");
        }
    })
    .permitAll() //http 에 설정한 로그인 페이지에는 인증을 받지 않아도 접근할 수 있도록 함
```
* UsernamePasswordAuthenticationFilter는 form login 인증처리를 담당하는 필터이다.
* 동작 순서
  1. AntPathRequestMatcher가 loginProcessingUrl()로 변경된(변경하지 않았다면 /login) url로 로그인 요청이 왔는지를 검사한다.
  2. url이 일치한다면 전송된 username과 password를 가지고 Authentication 객체를 만들어 ProviderManager객체에 전달한다.
  3. ProviderManager는 form 인증처리를 담당하는 AuthenticationProvider에 Authentication객체를 전달하며 인증처리를 위임한다.(다양한 방식의 인증처리를 담당하는 AuthenticationProvider가 있고, 어떠한 인증을 수행하느냐에 따라 다른 AuthenticationProvider가 선택된다.)
  4. 인증에 성공한다면 AuthenticationProvider는 사용자 정보와 권한정보를 담은 Authentication 객체를 생성해 반환한다.
  5. UsernamePasswordAuthenticationFilter는 반환된 Authentication 객체를 SecurityContext 객체에 저장한다.
  6. SuccessHandler가 있다면 해당 핸들러에 정의된 로직을 실행한다.
## LogoutFilter
<center><img src="./../img/LogoutFilter.png"></center>

```java
http
    .logout() //로그아웃 처리 설정 시작
    .logoutUrl("/logout") //로그아웃 url, default 로 post 방식으로 "/logout"을 받아야한다.
    // .logoutSuccessUrl("/login") //로그아웃 성공시 리다이렉트될 url
    .addLogoutHandler(new LogoutHandler() { //LogoutHandler 는 인터페이스
        //로그아웃 처리시 실행할 핸들러를 추가해준다.
        //세션을 무효화시키는 핸들러 등이 기본적으로 있고, 그 외에도 추가적인 작업을 하고싶을 때 새로운 핸들러를 만들어 추가해준다.
        @Override
        public void logout(HttpServletRequest request, HttpServletResponse response, Authentication authentication) {
            System.out.println("logout success"); 
        }
    })
    .logoutSuccessHandler(new LogoutSuccessHandler() { //LogoutSuccessHandler 는 인터페이스

        //로그아웃 성공시 실행할 핸들러를 추가해준다.
        /*
        핸들러는 하나만 넣을 수 있다. 따라서 logoutSuccessUrl()은 해당 url 로 redirect 하는 핸들러를
        추가하는 메서드이기 때문에 내가 따로 핸들러를 만들어서 넣는다면 해당 핸들러에 redirect 로직을 넣어야 한다.
        */

        @Override
        public void onLogoutSuccess(HttpServletRequest request, HttpServletResponse response, Authentication authentication) throws IOException, ServletException {
            //로그인 페이지로 리다이렉트
            response.sendRedirect("/login");
        }
    })
    .deleteCookies("JSESSIONID", "remember-me") //해당하는 이름의 쿠키를 삭제(JSESSIONID, remember-me 쿠키 삭제)
```
```java
@GetMapping("/logout")
public ResponseEntity<String> logout(HttpServletRequest request, HttpServletResponse response) {

    Authentication authentication = SecurityContextHolder.getContext().getAuthentication();

    if (authentication != null) {
        new SecurityContextLogoutHandler().logout(request, response, authentication);
        return new ResponseEntity<>(HttpStatus.OK);
    }

    return new ResponseEntity<>(HttpStatus.UNAUTHORIZED);
}
```
* LogoutFilter는 로그아웃 처리를 담당하는 필터이다.
* logout 필터는 기본적으로 post 방식으로 동작하기 때문에 get 방식으로 로그아웃 하기를 원한다면 컨트롤러에 따로 위와 같은 핸들러 메서드를 만들어줘야 한다.
* 동작순서
  1. AntPathRequestMatcher가 logoutUrl()로 변경된(변경하지 않았다면 /logout) url로 로그아웃 요청이 왔는지를 검사한다.
  2. url이 일치한다면 SecurityContext로부터 해당 사용자의 정보가 담긴 Authentication 객체를 받는다.
  3. LogoutHandler가 해당 Authentication 객체를 받아 세션을 무효화하고 쿠키를 삭제하는 등의 처리를 한다.(LogoutHandler는 하나가 아닌 여러개가 있고 내가 추가할 수도 있다.)
```java
@Controller
public class LogoutController {

    @GetMapping("/logout")
    public String logout(HttpServletRequest request, HttpServletResponse response) {

        Authentication authentication = SecurityContextHolder.getContext().getAuthentication();

        if (authentication != null) {
            new SecurityContextLogoutHandler().logout(request, response, authentication);
        }
        return "redirect:/";
    }
}
```
* post 방식이 아닌 get 방식으로 로그아웃 하기위해서 컨트롤러에서 get요청을 받아 SercurityContextLogoutHandler 를 통해 로그아웃 처리를 할 수 있다.
## RememberMeAuthenticationFilter
<center><img src="./../img/RememberMeAuthenticationFilter.png"></center>

```java
@Bean
public PersistentTokenRepository tokenRepository() {
    // JDBC 기반의 tokenRepository 구현체
    JdbcTokenRepositoryImpl jdbcTokenRepository = new JdbcTokenRepositoryImpl();
    jdbcTokenRepository.setDataSource(dataSource); // dataSource 주입
    return jdbcTokenRepository;
}
```
```java
public class JdbcTokenRepositoryImpl extends JdbcDaoSupport implements PersistentTokenRepository {

	/** Default SQL for creating the database table to store the tokens */
	public static final String CREATE_TABLE_SQL = "create table persistent_logins (username varchar(64) not null, series varchar(64) primary key, "
			+ "token varchar(64) not null, last_used timestamp not null)";

    ...

}
```
```java
@Table(name = "persistent_logins")
@Entity
@Getter
@Setter
public class PersistentLogins {

    @Id
    @Column(length = 64)
    private String series;

    @Column(nullable = false, length = 64)
    private String username;

    @Column(nullable = false, length = 64)
    private String token;

    @Column(name = "last_used", nullable = false, length = 64)
    private LocalDateTime lastUsed;

}
```
```java
http
    .rememberMe() //remember-em 설정 시작
    .rememberMeParameter("remember") //remember me 쿠키를 사용할지 말지를 정하는 파라미터(체크박스에 체크시 true)의 이름을 정함, 기본 파라미터명은 remember-me
    .tokenValiditySeconds(3600) //remember me 쿠키 만료 시간, default 는 14일(1209600초)
    // .alwaysRemember(true) //remember me 쿠키를 사용한다는 체크박스에 체크하지 않더라도 항상 remember me 쿠키 사용, default 는 false, 잘 사용하지 않는다.
    .tokenRepository(tokenRepository()) //PersistentTokenRememberMeServices 사용
    .userDetailsService(userDetailsService); //필수, 사용할 UserDetailsService를 정한다.
```
* RememberMeAuthenticationFilter는 remember-me 쿠키에 대한 처리를 담당하는 처리이다. remember-me쿠키는 세션이 만료되어도 계속해서 로그인이 되어있을 수 있도록 해주는 쿠키이다.
* remember-me 쿠키에는 사용자의 username 정보, password 정보, 쿠키 만료일 등이 인코딩되어 들어있다.
* TokenBasedRememberMeServices를 사용할 경우 remember-me 쿠키는 username, password, 만료기한, 토큰(랜덤값, 인증할 때마다 바뀜)이 인코딩되어 만들어진다. 하지만 이러한 방식으로 쿠키 값을 정하면 쿠키 값을 탈취당할 경우 해커는 탈취한 쿠키로 계속해서 인증할 수 있고 희생자는 인증을 할 수 없게 된다.(해커가 인증을 하면서 새로 바뀐 쿠키값을 가지고 갔기 때문) 이러한 문제를 해결하기 위해서 PersistentTokenBasedRememberMeServices를 사용한다. PersistentTokenBasedRememberMeServices를 사용하면 remember-me 쿠키는 username, password, 만료기한, 토큰(랜덤 값, 인증할 때마다 바뀜), 시리즈(랜덤 값, 고정)이 인코딩되어 만들어진다. 그리고 해커에게 쿠키 값을 탈취당한 경우 희생자가 remember-me 쿠키로 인증을 시도했을 때 유효하지 않은 토큰 값과 유효한 시리즈 값을 가지고 인증을 시도한다. 이런 경우 서버는 모든 토큰을 삭제하여 해커가 더 이상 탈취한 쿠키를 사용하지 못하도록 방지한다.
* PersistentTokenBasedRememberMeServices를 사용하려면 PersistentTokenRepository를 스프링 빈으로 등록해야 한다. PersistentTokenRepository는 인터페이스이고 JdbcTokenRepositoryImpl은 구현체이다. 그리고 JdbcTokenRepositoryImpl의 소스코드를 보면 `CREATE_TABLE_SQL`에 사용자 정보와 토큰 값을 저장하는 테이블을 생성하는 쿼리문이 있다. 이 테이블을 사용하기 위해서 이 테이블에 매핑되는 엔티티를 만들어 줄 필요가 있다.
* 동작 순서
  1. 해당 사용자의 Authentication값이 null일 경우(세션이 만료되거나 끊어져서 해당 세션의 SecurityContext를 찾지 못할 경우)에 RememberMeAuthenticationFilter가 동작하고 그렇지 않으면 다음 필터로 넘어간다.
  2. RememberMeServices는 인터페이스이고 TokenBasedRememberMeServices와 PersistentTokenBasedRememberMeServices는 구현체이다. .tokenRepository()가 지정되어있는 경우, request를 PersistentTokenBasedRememberMeServices에 넘겨주어 인증처리를 하고 그렇지 않은 경우, request를 TokenBasedRememberMeServices에 쿠키를 넘겨주어 인증처리를 한다. 
  3. request를 넘겨받은 RememberMeServices는 request로부터 remember-me 쿠키 값을 추출한다. 만약 만약 추출한 remember-me 쿠키 값이 null이 아니라면 cookieTokens(remember-me 쿠키 값을 디코딩 한 값)을 추출한다.
  4. 토큰이 정상적인 토큰인지(정상적인 포맷을 지키고 있는지) 검사하고 정상적인 토큰이면 해당 토큰이 서버에도 있는지 검사한다. 이 때 TokenBasedRememberMeServices의 경우, 서버는 클라이언트에게 받은 토큰 값을 서버의 메모리에 저장된 토큰 값과 비교한다. PersistentTokenBasedRememberMeServices의 경우, 서버는 클라이언트에게 받은 토큰 값을 서버의 DB에 저장된 토큰 값과 비교한다.
  5. 만약 토큰이 존재한다면 토큰에서 사용자 정보를 추출하여 Authentication 객체를 만들어 해당 객체를 AuthenticationManager에 넘겨주어 인증처리를 시작한다.
## AnonymousAuthenticationFilter
<center><img src="./../img/AnonymousAuthenticationFilter.png"></center>

* AnonymousAuthenticationFilter는 해당 요청이 인증되지 않은 사용자의 요청일 때 동작하는 필터이다.
* 동작 순서
  1. SecurityContext로부터 세션 값에 매핑되는 Authentication 객체가 있는지 검사한다.
  2. 만약 Authentication 객체가 없다면 AnonymousAuthenticationToken을 생성해 해당 객체를 SecurityContext에 저장한다. AnonymousAuthenticationToken은 인증되지 않은 사용자에 대한 Authentication 객체이다.(Authentication은 인터페이스이고 AnonymousAuthenticationToken은 Authentication을 구현한 구현체이다)
## SessionManagementFilter, ConcurrentSessionFilter
### 동시 세션(로그인) 제어 전략
<center><img src="./../img/SessionPolicy.png"></center>

* 동시 세션 제어 전략에는 2가지 방식이 있다. 이미 로그인된 아이디로 다른 사용자가 로그인을 시도했을 때 이전 사용자의 세션을 만료시키는 전략, 그리고 이미 로그인된 아이디로 다른 사용자가 로그인을 시도했을 때 현재 로그인을 시도하는 사용자의 인증을 실패 처리하는 전략이다.
* maxSessionsPreventsLogin(true) : 이미 로그인된 아이디로 다른 사용자가 로그인을 시도했을 때 현재 로그인을 시도하는 사용자의 인증을 실패 처리하는 전략
* maxSessionsPreventsLogin(false) : 이미 로그인된 아이디로 다른 사용자가 로그인을 시도했을 때 이전 사용자의 세션을 만료시키는 전략(default)

### 필터 동작 방식, 세션 제어 설정 API
<center><img src="./../img/SessionFilter.png"></center>

```java
http
    .sessionManagement() //동시 세션제어 설정 시작
    .invalidSessionUrl("/invalid") //세션이 유효하지 않을 때 이동할 url
    .maximumSessions(1) //최대 혀용 가능 세션 수, -1일 경우 같은 아이디로 무제한 로그인 세션 허용
    /*
    최대 세션 혀용 개수를 초과했을 경우에 어떤 세션 전략을 사용할지 설정함
    true 면 현재 로그인된 사용자가 있을 시 새로 로그인하는 사용자에게 인증 예외를 전달
    false 면 현재 로그인 된 사용자가 있을 시 현재 로그인된 사용자의 세션을 만료시키고 새로 로그인하는 사용자의
    세션을 생성함(default : false)
    */
    .maxSessionsPreventsLogin(true)
    .expiredUrl("/expired") // 세션이 만료된 경우 이동할 url
    .and()
    /*
    SessionCreationPolicy.ALWAYS : 스프링 시큐리티가 항상 세션 생성
    SessionCreationPolicy.IF_REQUIRED : 스프링 시큐리티가 필요시 세션 생성(default)
    SessionCreationPolicy.NEVER : 스프링 시큐리티가 세션을 생성하지 않지만 이미 존재하면 사용
    SessionCreationPolicy.STATELESS : 스프링 시큐리티가 세션을 생성하지도 않고 존재해도 사용하지 않음(JWT 인증 방식을 사용할 때 사용)
        */
    .sessionCreationPolicy(SessionCreationPolicy.IF_REQUIRED)
    .sessionFixation().changeSessionId(); //세션 고정 취약점 공격을 막는다. 요청마다 다른 세션키를 사용(스프링 3.1버전 이상)(default)
    // .sessionFixation().none(); //항상 같은 세션키를 사용
    // .sessionFixation().migrageSession(); //changeSessionId()와 기능은 같다.(스프링 3.1미만에서 사용)
    // .sessionFixation().newSession(); //세션키값만 바꾸는게 아니라 세션 자체를 요청마다 새로 생성한다.(이전에 사용하던 세션 속성 값이 세션에 저장되지 않음)
```
```java
// @Bean
// public SessionRegistry sessionRegistry() {
//     return new SessionRegistryImpl();
// }

//이것만 등록했는데 안될 경우 위에 것도 스프링 빈으로 등록하자
@Bean
public static ServletListenerRegistrationBean<HttpSessionListener> httpSessionEventPublisher() {
    return new ServletListenerRegistrationBean<>(new HttpSessionEventPublisher());
}
```
* SessionManagementFilter : 사용자가 로그인을 시도했을 때 동작하는 필터이다.
* ConcurrentSessionFilter : 이미 인증된 사용자가 요청을 보냈을 때 동작하는 필터이다.
* HttpSessionEventPublisher를 서블릿 리스너로 등록하는 이유 : 사용자가 로그인을 하면 SessionRegistry 객체에 있는 Map에 세션정보를 저장하게 된다. 이 때 key는 sessionId가 된다. 그리고 사용자가 로그아웃을 할 경우 LogoutFilter가 동작하며 해당 세션을 invalidate 해주지만 SessionRegistry에 있는 Map에 저장된 해당 세션 정보를 삭제하지는 않는다. 그래서 다시 같은 아이디로 로그인을 하면 해당 아이디의 principal을 가진 세션이 삭제되지 않고 Map에 들어있기 때문에 최대 허용 가능 세션 수를 초과하지 않았더라도 초과한 것으로 판단된다.(쉽게 말하면 나는 로그아웃 해서 세션 자체은 invalidate 됐지만 동시 세션 제어에서는 로그인 되어있는 것으로 판단됨) 이를 피하기 위해서는 HttpSessionEventPublisher를 서블릿 리스너로 등록해야한다.(HttpSessionEventPublisher는 HttpSessionListener의 구현체) 로그아웃을 하면서 세션이 invalidate 되면 HttpSessionEventPublisher의 sessionDestroyed(HttpSessionEvent event) 메서드가 호출되고 HttpSessionDestroyedEvent를 발생시킨다. SessionRegistry의 구현체인 SessionRegistryImpl은 ApplicationListener\<SessionDestroyedEvent> 인터페이스를 구현하고 있기 때문에 HttpSessionDestroyedEvent가 발생하면 onApplicationEvent(SessionDestroyedEvent event) 메서드가 호출되고 여기에서 removeSessionInformation(sessionId)를 실행하면서 Map에 저장된 세션 정보를 삭제하게 된다.
* SessionManagementFilter 동작 순서
  1. 사용자가 로그인을 시도하면 최대 세션 허용 개수가 초과되었는지 검사한다.
  2. 최대 세션 허용 개수가 초과되었다면 maxSessionsPreventsLogin(true)일 경우에는 현재 로그인을 시도하는 사용자의 인증을 실패 처리하고 maxSessionsPreventsLogin(false)일 경우에는 이전 사용자의 세션을 만료시킨다.
* ConcurrentSessionFilter 동작 순서
  1. 이미 인증된 사용자가 서버에 리소스를 요청한 경우에 해당 세션이 만료되었는지를 검사한다.
  2. 세션이 만료되었다면 서버는 로그아웃 처리를 수행한 후 오류 페이지를 응답한다.
## ExceptionTranslationFilter
<center><img src="./../img/ExceptionTranslationFilter.png"></center>

```java
http.exceptionHandling()
    .authenticationEntryPoint(new AuthenticationEntryPoint() {
        @Override
        public void commence(HttpServletRequest request, HttpServletResponse response, AuthenticationException authException) throws IOException, ServletException {
            //인증 예외 발생 시(익명 사용자가 인가되지 않은 리소스에 접근했을 때) 실행할 로직을 넣는다.

            response.sendRedirect("/login");
        }
    })
    .accessDeniedHandler(new AccessDeniedHandler() {
        @Override
        public void handle(HttpServletRequest request, HttpServletResponse response, AccessDeniedException accessDeniedException) throws IOException, ServletException {
            //인가 예외 발생 시 실행할 로직을 넣는다.

            response.sendRedirect("/denied");
        }
    })
    .and().formLogin()
    .successHandler((request, response, authentication) -> {
            RequestCache requestCache = new HttpSessionRequestCache(); //session 에 savedRequest 를 저장하고 가져오는 역할을 하는 객체
            SavedRequest savedRequest = requestCache.getRequest(request, response); //savedRequest 를 가지고옴
            response.sendRedirect(savedRequest.getRedirectUrl()); //savedRequest 의 url 로 redirect
    })
```
* 인증 예외(인증받지 않는 사용자(익명 사용자)가 인가 예외를 발생시킨 경우)가 발생하여 로그인 페이지로 redirect 됬을 때, 로그인 성공 후 인증 예외가 발생한 url로 바로 이동하도록 하고싶으면 SuccessHandler에 해당 페이지로 redirect하는 로직을 넣어줘야 한다.
* FilterSecurityInterceptor에서는 AuthenticationException과 AccessDeniedException을 throw한다. 이 때 ExceptionTranslationFilter가 AuthenticationEntryPoint의 commence를 호출하는 경우는 AuthenticationException가 throw됐을 때(Authentication 객체가 null일 경우)가 아니라 익명사용자가 AccessDeniedException을 발생시킨 경우이다. 익명사용자가 인가되지 않은 리소스에 접근해 FilterSecurityInterceptor에서 AccessDeniedException가 throw되면 ExceptionTranslationFilter에서 AccessDeniedException을 AuthenticationException으로 바꾼다.
* 인증 예외 시 동작
  1. FilterSecurityInterceptor에서 인증 예외가 발생하면(익명사용자가 AccessDeniedException을 발생시킨 경우) ExceptionTranslationFilter는 AuthenticationEntryPoint를 구현한 구현체의 commence 메서드를 실행한다.(일반적으로 로그인 페이지로 redirect 한다.) 이 때 AuthenticationException을 만들어 commence의 파라미터로 넣는다.
  2. 인증 예외가 발생한 request의 정보(url 정보, 헤더 정보, 파라미터 정보 등)를 SavedRequest에 담아(일반적으로 DefaultSavedRequest 구현체에 담는다.) Session에 저장한다.(Session에 저장하는 로직이 HttpSessionRequestCache에 구현되어 있다.)
* 인가 예외 시 동작
  1. FilterSecurityInterceptor에서 인가 예외가 발생하면 ExceptionTranslationFilter는 AccessDeniedHandler를 구현한 구현체의 handle 메서드를 실행한다.(일반적으로 예외페이지로 redirect 한다.)
## CSRF Filter
<img src="./../img/csrf.png">

* CSRF 필터는 모든 : PATCH, POST, PUT, DELETE 메소드 요청에 대해 랜덤하게 생성된 토큰을 HTTP 파라미터로 요구한다.
* 요청 시 전달되는 토큰 값과 서버에 저장된 실제 값과 비교한 후 만약 일치하지 않으면 요청은 실패한다.
```javascript
const MyTheme = {
  color: {
    bgColor: "#f3efe4",
    textColor: "#665c54",
    borderColor: "#928374",
    clickColor: "#689d6a",
    navColor: "#1d2021",
    navTextColor: "white",
    btnColor: "#645943",
    tableBorderColor: "#dadfec",
    tableBgColor: "#ecf0fa",
  },
  font: "Noto Sans CJK KR"
}

const root = ReactDOM.createRoot(
  document.getElementById('root') as HTMLElement
);
root.render(
  <React.StrictMode>
      <ThemeProvider theme={MyTheme}>
        <App />
      </ThemeProvider>
  </React.StrictMode>
);

Modal.setAppElement('#root');
// csrf 토큰 설정
axios.defaults.xsrfCookieName = 'XSRF-TOKEN';
axios.defaults.xsrfHeaderName = 'X-XSRF-TOKEN';
```
```java
@Configuration
@EnableWebSecurity
@RequiredArgsConstructor
public class SecurityConfig extends WebSecurityConfigurerAdapter {

    ...

    @Override
    protected void configure(HttpSecurity http) throws Exception {
        ...

        http.csrf()
                .csrfTokenRepository(CookieCsrfTokenRepository.withHttpOnlyFalse());
    }
}
```
* 리액트와 스프링을 함께 사용할 때 csrf 토큰을 사용하고 싶으면 위와 같이 설정하면 된다.
## FilterSecurityInterceptor
<img src="./../img/filterSecurityInterceptor.png">

* FilterSecurityInterceptor는 자원에 대한 인가를 담당하는 필터이고, 스프링 시큐리티의 필터 중 가장 마지막에 위치한다.
* 동작 순서
  1. FilterSecurityInterceptor에 전달된 Authentication 객체가 null이면 ExceptionTranslationFilter에 AuthenticationException을 throw한다.
  2. Authentication 객체가 null이 아니면 SecurityMetaDataSource는 해당 자원에 대한 권한 정보를 조회 한다. 이 때 권한 정보는 Map의 형태로 저장되어 있는데 만약 사용자가 접근하는 자원에 대한 권한 정보가 null일 경우 자원 접근을 허용한다.
  3. 사용자가 접근하는 자원에 대한 권한 정보가 존재 한다면 AccessDecisionManager에게 현재 사용자가 해당 자원에 접근할 수 있는지 판단하도록 인가 처리를 위임한다.
  4. 만약 접근이 허가 되지 않으면 AccessDeniedException을 ExceptionTranslationFilter에 throw 하고 그렇지 않으면 사용자는 최종적으로 자원에 접근하게 된다.
## 권한 설정 API, SecurityMetadataSource
```java
http.authorizeRequests() //http request 인가 시작
    .antMatchers("/denied").permitAll()
    .antMatchers("/user").hasRole("USER") // "/user" url 에 접근하려면 USER 라는 role 을 가져야함
    .antMatchers("/admin/pay").hasRole("ADMIN") //"/admin/pay" url 에 접근하려면 ADMIN 라는 role 을 가져야함
    .antMatchers("/admin/**").access("hasRole('ADMIN') or hasRole('SYS')") // "/admin" 하위의 url(/pay 제외)에 접근하려면 ADMIN or SYS 라는 role 을 가져야함
    .anyRequest().authenticated(); //모든 request 에 대해 (인가에 대한)인증을 수행함
```
* ant 패턴의 표현식을 통해 특정 경로의 url에 대한 권한설정이 가능한다.
* 자원에 대한 권한은 SecurityMetadataSource에 Map의 형태로 저장된다.(DefaultFilterInvocationSecurityMetadataSource 구현체의 requestMap)
* 이렇게 SecurityMetadataSource에 저장된 자원에 대한 권한으로부터 FilterSecurityInterceptor는  List\<ConfigAttribute>를 반환받아 AccessDecisionManager에 전달하게 된다.
* 권한 설정은 설정한 순서대로 진행되기 때문에 구체적인 경로가 먼저오고 그것보다 큰 범위의 경로가 뒤에 오도록 해야한다.(만약 앞쪽의 권한 설정을 통과한다면 뒤쪽의 권한 설정은 진행하지 않는다.)
* 인가 API
  + authenticated() : 인증된 사용자의 접근만을 허용
  + fullyAuthenticated() : 인증된 사용자의 접근만을 허용, rememberMe 인증 제외
  + permitAll() : 무조건 접근을 허용
  + denyAll() : 무조건 접근을 허용하지 않음
  + anonymous() : 익명사용자의 접근만을 허용(다른 사용자 접근 안됨)
  + rememberMe() : remember-me를 통해 인증된 사용자의 접근만을 허용
  + access(String) : 주어진 SpEL 표현식의 결과가 true이면 접근을 허용
  + hasRole(String) : 사용자가 주어진 역할이 있다면 접근을 허용
  + hasAuthority(String) : 사용자가 주어진 권한이 있다면 접근을 허용 __(hasRole()과의 차이점은 hasRole()의 경우에는 자동으로 앞에 "ROLE_"을 붙여서 DB에서 해당 사용자의 권한을 찾음 예를들면 hasRole("USER")를 하면 해당 사용자가 "ROLE_USER"라는 권한을 갖고 있는지를 찾음 하지만 hasAuthority("USER")를 하면 사용자가 "USER"라는 권한을 갖고 있는지를 찾음)__
  + hasAnyRole(String...) : 사용자가 주어진 역할이 있다면 접근을 허용
  + hasAnyAuthority(String...) : 사용자가 주어진 권한 중 어떤 것이라도 있다면 접근을 허용
  + hasIpAddress(String) : 주어진 IP로부터 요청이 왔다면 접근을 허용
## AccessDecisionManager, AccessDecisionVoter
* 요청에 대한 자원 접근을 허용할지 거부할지를 결정하는 객체이다. AccessDecisionManager는 여러개의 AccessDecisionVoter를 가지고 있다.
* AccessDecisionVoter는 인터페이스이며 여러개의 구현체를 가지고 있고, 각 구현체는 Authentication(사용자 정보), FilterInvocation(요청 자원 정보), List\<ConfigAttribute>(해당 자원에 대한 권한 정보들)를 받아 접근을 허용할지 말지 판단한다.
* AccessDecisionManager는 인터페이스로 AffirmativeBased, ConsensusBased, UnanimousBased 구현체가 있다.
  - AffirmativeBased : 여러 AccessDecisionVoter 객체 중 하나라도 접근을 허가하면 AccessDecisionVoter.ACCESS_GRANTED를 반환한다.
  - ConsensusBased : 다수결에 의해 판단한다. 동수일 경우에는 allowIfEqualGrantedDeniedDecision 프로프티 값이 true면 AccessDecisionVoter.ACCESS_GRANTED를 반환하고(default), false면 AccessDecisionVoter.ACCESS_DENIED를 반환한다.
  - UnanimousBased : 모든 AccessDecisionVoter가 접근을 허가해야 AccessDecisionVoter.ACCESS_GRANTED를 반환하고 그렇지 않을 경우 AccessDecisionVoter.ACCESS_DENIED를 반환한다.