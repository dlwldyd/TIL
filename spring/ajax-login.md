# JSON 포맷 로그인 처리
## 커스텀 AuthentiactionFilter(JSON 포맷 인증 처리를 하는 필터)
```java
public class AjaxLoginProcessingFilter extends AbstractAuthenticationProcessingFilter {

    //ajax 는 json 으로 데이터를 보내기 때문에 ObjectMapper 를 통해 json 을 객체로 변환해줄 필요가 있음
    private ObjectMapper objectMapper = new ObjectMapper();

    public AjaxLoginProcessingFilter() {
        // "/api/login"으로 요청이 들어오면 필터가 작동함
        super(new AntPathRequestMatcher("/api/login"));
    }

    @Override
    public Authentication attemptAuthentication(HttpServletRequest request, HttpServletResponse response) throws AuthenticationException, IOException, ServletException {

        if (!isAjax(request)) {
            throw new IllegalStateException("Authentication is not supported");
        }

        AccountDto accountDto = objectMapper.readValue(request.getReader(), AccountDto.class);
        if (!(hasText(accountDto.getUsername()) && hasText(accountDto.getPassword()))) {
            throw new IllegalArgumentException("Username or Password is empty");
        }

        //인증 처리를 위한 Authentication 토큰 생성
        AjaxAuthenticationToken authentication = new AjaxAuthenticationToken(accountDto.getUsername(), accountDto.getPassword());

        //AuthenticationManager 에게 인증처리 위임
        return getAuthenticationManager().authenticate(authentication);
    }

    private boolean isAjax(HttpServletRequest request) {
        //JSON 포맷인지 구분하는 특정한 방법이 있는게 아니라 그냥 클라이언트 측과의 약속을 통해 헤더에 JSON 포맷인지를 나타내는 정보를 덧붙여서 보냄

        //"X-Requested-with" 헤더 값이 "XMLHttpRequest"면 로그인 처리를 함
        if ("XMLHttpRequest".equals(request.getHeader("X-Requested-with"))) {
            return true;
        }

        return false;
    }
}
```
```java
/*
JSON 포맷 인증 처리를 위한 토큰, 커스텀 Authentication 토큰을 만들기 위해서는 AbstractAuthenticationToken을 상속받아야 한다.
Authentication 을 구현해도 되지만 전부 다 구현하기에는 시간이 너무 많이 걸린다.
*/
public class AjaxAuthenticationToken extends AbstractAuthenticationToken {

    private static final long serialVersionUID = SpringSecurityCoreVersion.SERIAL_VERSION_UID;

    private final Object principal;

    private Object credentials;

    //인증 이전 사용되는 생성자
    public AjaxAuthenticationToken(Object principal, Object credentials) {
        super(null);
        this.principal = principal;
        this.credentials = credentials;
        setAuthenticated(false);
    }

    //인증 이후 사용되는 생성자
    public AjaxAuthenticationToken(Object principal, Object credentials,
                                               Collection<? extends GrantedAuthority> authorities) {
        super(authorities);
        this.principal = principal;
        this.credentials = credentials;
        super.setAuthenticated(true); // must use super, as we override
    }

    @Override
    public Object getCredentials() {
        return this.credentials;
    }

    @Override
    public Object getPrincipal() {
        return this.principal;
    }

    @Override
    public void setAuthenticated(boolean isAuthenticated) throws IllegalArgumentException {
        Assert.isTrue(!isAuthenticated,
                "Cannot set this token to trusted - use constructor which takes a GrantedAuthority list instead");
        super.setAuthenticated(false);
    }

    @Override
    public void eraseCredentials() {
        super.eraseCredentials();
        this.credentials = null;
    }
}
```
```java
@Configuration
@EnableWebSecurity
@RequiredArgsConstructor
public class AjaxSecurityConfig extends WebSecurityConfigurerAdapter {  

    //내가 만든 필터 스프링 빈으로 등록
    @Bean
    public AjaxLoginProcessingFilter ajaxLoginProcessingFilter() throws Exception {
        AjaxLoginProcessingFilter ajaxLoginProcessingFilter = new AjaxLoginProcessingFilter();
        //ajaxLoginProcessingFilter 에서 인증처리를 위임할 AuthenticationManager 를 설정해야한다.
        ajaxLoginProcessingFilter.setAuthenticationManager(authenticationManagerBean());
        return ajaxLoginProcessingFilter;
    }
    
    //AuthenticationManager 를 얻어오는 메서드
    @Override
    public AuthenticationManager authenticationManagerBean() throws Exception {
        return super.authenticationManagerBean();
    }

    @Override
    protected void configure(HttpSecurity http) throws Exception {

        http
                //UsernamePasswordAuthenticationFilter 앞에 AjaxLoginProcessingFilter 추가
                .addFilterBefore(ajaxLoginProcessingFilter(), UsernamePasswordAuthenticationFilter.class);

                //가장 마지막에 필터 추가
//                .addFilter(ajaxLoginProcessingFilter())

                //UsernamePasswordAuthenticationFilter 뒤에 AjaxLoginProcessingFilter 추가
//                .addFilterAfter(ajaxLoginProcessingFilter(), UsernamePasswordAuthenticationFilter.class)

                //UsernamePasswordAuthenticationFilter 를 AjaxLoginProcessingFilter 로 replace 한다.
//                .addFilterAt(ajaxLoginProcessingFilter(), UsernamePasswordAuthenticationFilter.class)
    }

    ...

}
```
* AbstractAuthenticationProcessingFilter를 상속받아 커스텀 AuthenticationFilter를 만들 수 있다.
* 생성자에서 어느 URL이 들어왔을 때 필터가 작동할지 정할 수 있다.
* attemptAuthentication(HttpServletRequest request, HttpServletResponse response) 메서드를 오버라이드 하여 인증 처리를 하는 로직을 만들 수 있다. 일반적으로 실제로 인증을 처리하는 로직은 AuthenticationProvider에 구현하고 필터는 AuthenticationManager에 인증처리를 위임한다.(AuthenticationManager가 인증처리를 할 AuthenticationProvider들을 가지고 있다.)
* 커스텀 Authentication 토큰을 만들기 위해서는 AbstractAuthenticationToken을 상속받아야 한다. Authentication을 구현해도 되지만 그렇게 되면 구현할 메서드가 너무 많아진다. JSON 포맷 인증 처리를 하는 AuthenticationProvider를 만들 때 커스텀 Authentication 토큰을 만들지 않고  UsernamePasswordAuthenticatinToken을 해당 AuthenticationProvider가 support 하도록 만들어도 된다.
* 내가 만든 필터를 등록할 때 setAuthenticationManager() 메서드를 통해 해당 필터가 위임할 AuthenticationManager를 세팅해야한다. AuthenticationManager는 WebSecurityConfigurerAdapter의 authenticationManagerBean() 메서드를 통해 얻을 수 있다.(로직을 바꿀 필요가 없으면 굳이 오버라이드할 필요는 없다.)
* addFilterBefore(), addFilter(), addFilterAfter(), addFilterAt() 메서드를 통해 내가 만들 필터의 위치를 지정해서 필터로 추가할 수 있다.
## 커스텀 AuthenticationProvider(JSON 포맷 인증 처리를 하는 프로바이더)
```java
@RequiredArgsConstructor
public class AjaxAuthenticationProvider implements AuthenticationProvider {

    private final UserDetailsService userDetailsService;
    private final PasswordEncoder passwordEncoder;

    //인증에 대한 검증 로직이 들어감
    @Override
    public Authentication authenticate(Authentication authentication) throws AuthenticationException {
        //authentication 에는 사용자가 입력한 아이디, 패스워드 정보가 들어있음
        String username = authentication.getName();
        String password = (String) authentication.getCredentials();

        AccountContext accountContext = (AccountContext) userDetailsService.loadUserByUsername(username);

        if (!passwordEncoder.matches(password, accountContext.getPassword())) {
            throw new BadCredentialsException("BadCredentialsException");
        }

        //인자가 2개인(권한 정보X) 생정자는 로그인 시도 시 사용되는 생성자이다.
        //인자가 3개인(권한 정보O) 생성자는 로그인 성공 시 인증 토큰을 만들기 위해 사용되는 생성자이다.
        //인자 : principal -> 사용자 정보, credential -> 패스워드 정보, authorities -> 권한 정보
        return new AjaxAuthenticationToken(
                accountContext.getAccount(),
                null,
                accountContext.getAuthorities());
    }

    //인증 토큰을 해당 provider 가 인증처리를 지원하는지
    @Override
    public boolean supports(Class<?> authentication) {
        return AjaxAuthenticationToken.class.isAssignableFrom(authentication);
    }
}
```
```java
@Configuration
@EnableWebSecurity
@RequiredArgsConstructor
@Order(0)
public class AjaxSecurityConfig extends WebSecurityConfigurerAdapter {

    private final UserDetailsService userDetailsService;
    private final PasswordEncoder passwordEncoder;

    @Bean
    public AuthenticationProvider ajaxAuthenticationProvider(UserDetailsService userDetailsService, PasswordEncoder passwordEncoder) {
        return new AjaxAuthenticationProvider(userDetailsService, passwordEncoder);
    }

    @Override
    protected void configure(AuthenticationManagerBuilder auth) throws Exception {
        auth.authenticationProvider(ajaxAuthenticationProvider(userDetailsService, passwordEncoder));
    }

    ...

}
```
* 일반적인 커스텀 프로바이더를 만들 때와 마찬가지로 JSON 포맷 인증 처리를 하는 프로바이더를 만들면 된다. 단, supports() 메서드에 Ajax, REST API 인증 처리를 하는 필터가 전달하는 토큰을 support 하도록 로직을 짜야한다.
## 커스텀 AuthenticationSuccessHandler(인증 성공), AuthenticationFailureHandler(인증 실패)
```java
public class AjaxAuthenticationSuccessHandler implements AuthenticationSuccessHandler {

    private ObjectMapper objectMapper = new ObjectMapper();

    @Override
    public void onAuthenticationSuccess(HttpServletRequest request, HttpServletResponse response, Authentication authentication) throws IOException, ServletException {
        Account account = (Account) authentication.getPrincipal();

        response.setStatus(HttpServletResponse.SC_OK);
        response.setContentType(MediaType.APPLICATION_JSON_VALUE);

        objectMapper.writeValue(response.getWriter(), account);
    }
}
```
```java
public class AjaxAuthenticationFailureHandler implements AuthenticationFailureHandler {

    private ObjectMapper objectMapper = new ObjectMapper();

    @Override
    public void onAuthenticationFailure(HttpServletRequest request, HttpServletResponse response, AuthenticationException exception) throws IOException, ServletException {

        String errorMessage = "Invalid Username or Password";

        response.setStatus(HttpServletResponse.SC_UNAUTHORIZED);
        response.setContentType(MediaType.APPLICATION_JSON_VALUE);

        if (exception instanceof DisabledException) {
            errorMessage = "Locked";
        } else if (exception instanceof CredentialsExpiredException) {
            errorMessage = "Expired Password";
        }

        objectMapper.writeValue(response.getWriter(), errorMessage);
    }
}
```
```java
@Configuration
@EnableWebSecurity
@RequiredArgsConstructor
@Order(0)
public class AjaxSecurityConfig extends WebSecurityConfigurerAdapter {

    @Bean
    public AjaxLoginProcessingFilter ajaxLoginProcessingFilter() throws Exception {
        AjaxLoginProcessingFilter ajaxLoginProcessingFilter = new AjaxLoginProcessingFilter();
        //ajaxLoginProcessingFilter 에서 인증처리를 위임할 AuthenticationManager 를 설정해야한다.
        ajaxLoginProcessingFilter.setAuthenticationManager(authenticationManagerBean());

        //핸들러는 여기서 추가해야함
        ajaxLoginProcessingFilter.setAuthenticationSuccessHandler(ajaxAuthenticationSuccessHandler());
        ajaxLoginProcessingFilter.setAuthenticationFailureHandler(ajaxAuthenticationFailureHandler());
        return ajaxLoginProcessingFilter;
    }

    ...

}
```
* REST API 로그인 처리 후 JSON 포맷으로 응답해야 하기 때문에 AuthenticationSuccessHandler와 AuthenticationFailureHandler에서 response body에 데이터를 write 하는 로직을 짜야 한다.
* 핸들러를 http.formLogin()에서 추가하면 UsernamePasswordAuthenticationFilter에 핸들러가 추가되기 때문에 내가만든 커스텀 필터의 setAuthenticationSuccessHandler() 메서드와 setAuthenticationFailureHandler() 메서드를 통해 핸들러를 추가해야한다.
## 커스텀 AuthenticationEntryPoint(익명사용자의 인가 예외), AccessDeniedHandler(인가 예외)
```java
public class AjaxLoginAuthenticationEntryPoint implements AuthenticationEntryPoint {

    //익명사용자가 인증이 필요한 리소스에 접근했을 때 수행하는 로직
    @Override
    public void commence(HttpServletRequest request, HttpServletResponse response, AuthenticationException authException) throws IOException, ServletException {

        response.sendError(HttpServletResponse.SC_UNAUTHORIZED, "UnAuthorized");
    }
}
```
```java
public class AjaxAccessDeniedHandler implements AccessDeniedHandler {

    //사용자가 권한 없이 리소스에 접근했을 때 수행하는 로직
    @Override
    public void handle(HttpServletRequest request, HttpServletResponse response, AccessDeniedException accessDeniedException) throws IOException, ServletException {

        response.sendError(HttpServletResponse.SC_FORBIDDEN, "Access is denied");
    }
}
```
```java
@Configuration
@EnableWebSecurity
@RequiredArgsConstructor
@Order(0)
public class AjaxSecurityConfig extends WebSecurityConfigurerAdapter {

    @Bean
    public AccessDeniedHandler ajaxAccessDeniedHandler() {
        return new AjaxAccessDeniedHandler();
    }

    @Bean
    public AuthenticationEntryPoint ajaxLoginAuthenticationEntryPoint() {
        return new AjaxLoginAuthenticationEntryPoint();
    }

    @Override
    protected void configure(HttpSecurity http) throws Exception {

        http.exceptionHandling()
                .authenticationEntryPoint(new AjaxLoginAuthenticationEntryPoint())
                .accessDeniedHandler(ajaxAccessDeniedHandler());

    }

    ...

}
```
* 인가 예외 시 JSON 포맷으로 응답해야 하기 때문에 AuthenticationEntryPoint AccessDeniedHandler에서 response body에 데이터를 write 하는 로직을 짜야 한다.
## CSRF 토큰
```html
<!DOCTYPE html>
<html xmlns:th="http://www.thymeleaf.org">

<meta id="_csrf" name="_csrf" th:content="${_csrf.token}"/>
<meta id="_csrf_header" name="_csrf_header" th:content="${_csrf.headerName}"/>

<script>
    function formLogin(e) {

        var username = $("input[name='username']").val().trim();
        var password = $("input[name='password']").val().trim();
        var data = {"username" : username, "password" : password};

        var csrfHeader = $('meta[name="_csrf_header"]').attr('content')
        var csrfToken = $('meta[name="_csrf"]').attr('content')

        $.ajax({
            type: "post",
            url: "/api/login",
            data: JSON.stringify(data),
            dataType: "json",
            beforeSend : function(xhr){
                xhr.setRequestHeader(csrfHeader, csrfToken);
                xhr.setRequestHeader("X-Requested-With", "XMLHttpRequest");
                xhr.setRequestHeader("Content-type","application/json");
            },
            success: function (data) {
                console.log(data);
                window.location = '/';
            },
            error : function(xhr, status, error) {
                console.log(error);
                window.location = '/login?error=true&exception=' + xhr.responseText;
            }
        });
    }
</script>

...


```
* 일반적인 form 로그인을 통해 로그인 할 때는 csrf 토큰이 자동으로 포함되지만(스프링 시큐리티가 자동으로 처리해줌) JSON 포맷을 통해 로그인할 때는 csrf 토큰이 자동으로 포함되지 않기 때문에 직접 csrf 토큰을 포함시켜 줘야한다.(CSRF 필터를 꺼도 되지만 별로 좋은 방법은 아니다.)