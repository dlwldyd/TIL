# Form Login 인증처리
## WebIgnore 설정(보안 예외 설정)
```java
@Configuration
@EnableWebSecurity
@RequiredArgsConstructor
public class SecurityConfig extends WebSecurityConfigurerAdapter {
 
    @Override
    public void configure(WebSecurity web) throws Exception {
        //webIgnore 설정, 정적 리소스들이 스프링 시큐리티 필터를 거치지 않는다.
        web.ignoring().requestMatchers(PathRequest.toStaticResources().atCommonLocations());
    }

    ...

}
```
* permitAll()은 필터를 거치면서 접근을 허가할지 검사하지만 WebIgnore 설정을 하면 보안필터 자체를 거치지 않는다. 그렇기 때문에 정적 리소스에 대한 접근은 WebIgnore 설정을 해서 보안필터를 거치지 않게 하는 것이 비용적으로 더 싸다.
## PasswordEncoder를 이용해 패스워드 암호화
```java
@Bean
public PasswordEncoder passwordEncoder() {
    return PasswordEncoderFactories.createDelegatingPasswordEncoder();
}
```
```java
@Service
@RequiredArgsConstructor
@Transactional(readOnly = true)
public class UserServiceImpl implements UserService {

    private final UserRepository userRepository;
    
    @Override
    @Transactional
    public void createUser(Account account) {
        userRepository.save(account);
    }
}
```
```java
@PostMapping("/users")
public String createUser(AccountDto accountDto) {
    userService.createUser(convertDtoToAccount(accountDto, passwordEncoder));
    return "redirect:/";
}
```
```java
//Account 클래스(엔티티)의 멤버 함수, AccountDto를 Account객체로 변환함
public static Account convertDtoToAccount(AccountDto accountDto, PasswordEncoder passwordEncoder) {
    return new Account(accountDto.getUsername(),
            passwordEncoder.encode(accountDto.getPassword()), //패스워드 암호화
            accountDto.getEmail(),
            accountDto.getAge(),
            accountDto.getRole());
}
```
* 패스워드를 데이터베이스에 저장할 때는 암호화를 할 필요가 있다. 암호화를 하기 위해서는 PasswordEncoder가 필요한데 PasswordEncoder는 PasswordEncoderFactories.createDelegatingPasswordEncoder()를 통해 반환받을 수 있다. 일반적으로 PasswordEncoder를 스프링 빈으로 등록해 사용한다.
## 커스텀 UserDetailsService(인증 처리시 사용되는 서비스 객체)
```java
@Getter
//User 는 UserDetails 를 구현하는 객체이다.
public class AccountContext extends User {

    private final Account account;

    //생성자, UserDetails에는 username, password, 권한 정보가 꼭 들어가야함
    //UserDetails를 직접 구현하면 username, password, 권한 정보 외에도 다른 정보도 넣을 수 있음(여기에서는 Account)
    public AccountContext(Account account, Collection<? extends GrantedAuthority> authorities) {
        super(account.getUsername(), account.getPassword(), authorities);
        this.account = account;
    }
}
```
```java
@Service
@RequiredArgsConstructor
public class CustomUserDetailsService implements UserDetailsService {

    private final UserRepository userRepository;

    @Override
    public UserDetails loadUserByUsername(String username) throws UsernameNotFoundException {
        Account findAccount = userRepository.findByUsername(username);

        if (findAccount == null) {
            throw new UsernameNotFoundException("UsernameNoFoundException");
        }

        //GrantedAuthority 는 인터페이스이고 SimpleGrantedAuthority 는 구현체이다.
        //SimpleGrantedAuthority 는 그냥 권한(String)과 그에 대한 EqualsAndHashCode, getter, toString 이 구현되어 있는 객체이다.(별거 없음)
        List<GrantedAuthority> roles = new ArrayList<>();
        roles.add(new SimpleGrantedAuthority(findAccount.getRole()));

        //UserDetails 의 구현체를 반환해야함
        /*
        UserDetails 에는 아이디, 권한, 비밀번호를 가져오는 메서드와
        계정이 잠겨있는지, 계정이 만료됐는지, 비밀번호가 만료됐는지, 사용할 수 없는 계정인지를 반환하는 메서드가 있다.
         */
        return new AccountContext(findAccount, roles);
    }
}
```
```java
@Configuration
@EnableWebSecurity
@RequiredArgsConstructor
public class SecurityConfig extends WebSecurityConfigurerAdapter {
    
    private final UserDetailsService userDetailsService;

    @Override
    protected void configure(AuthenticationManagerBuilder auth) throws Exception {
        
        auth.userDetailsService(userDetailsService); // 사용할 UserDetailsService 등록
    }

    ...

}
```
* 데이터베이스에서 유저의 정보를 가져와 인증처리를 하기 위해서는 UserDetailsService를 구현해야한다.
* UserDetailsService의 loadUserByUsername(String username)은 UserDetails 타입을 반환하는데 스프링 시큐리티가 제공하는 User 클래스를 그대로 사용해도 되고 UserDetails를 구현해서 내가 원하는 정보를 내가 만든 객체에 담을 수도 있다.
* auth.userDetailsService()를 통해 커스텀 UserDetailsService를 등록할 수 있다.
## 커스텀 AuthenticationProvider(인증을 처리하는 객체)
```java
@RequiredArgsConstructor
public class CustomAuthenticationProvider implements AuthenticationProvider {

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

        
        //뷰에서 히든 필드로 secretKey 라는 파라미터로 "secret"이라는 문자열을 넘기는데 만약 secretKey 라는 파라미터에 "secret"이라는 문자열이 없으면 예외를 발생시킴
        FormWebAuthenticationDetails details = (FormWebAuthenticationDetails) authentication.getDetails();
        String secretKey = details.getSecretKey();
        if (secretKey == null || !secretKey.equals("secret")) {
            throw new InsufficientAuthenticationException("InsufficientAuthenticationException");
        }

        //인자가 2개인(권한 정보X) 생정자는 로그인 시도 시 사용되는 생성자이다.
        //인자가 3개인(권한 정보O) 생성자는 로그인 성공 시 인증 토큰을 만들기 위해 사용되는 생성자이다.
        //인자 : principal -> 사용자 정보, credential -> 패스워드 정보, authorities -> 권한 정보
        return new UsernamePasswordAuthenticationToken(
                accountContext.getAccount(),
                null,
                accountContext.getAuthorities());
    }

    //인증 토큰을 해당 provider 가 인증처리를 지원하는지
    @Override
    public boolean supports(Class<?> authentication) {
        return UsernamePasswordAuthenticationToken.class.isAssignableFrom(authentication);
    }
}
```
```java
@Configuration
@EnableWebSecurity
@RequiredArgsConstructor
public class SecurityConfig extends WebSecurityConfigurerAdapter {
    
    private final UserDetailsService userDetailsService;

    @Bean
    public CustomAuthenticationProvider customAuthenticationProvider() {
        return new CustomAuthenticationProvider(userDetailsService, passwordEncoder());
    }
    
    @Override
    protected void configure(AuthenticationManagerBuilder auth) throws Exception {

        //auth.userDetailsService(userDetailsService); // customAuthenticationProvider가 userDetailsService를 주입받아 사용하고 있기 때문에 등록할 필요 없다.
        auth.authenticationProvider(customAuthenticationProvider());
    }

    ...

}
```
* AuthenticationProvider를 구현하여 아이디와 패스워드에 대한 검증 외에도 추가적인 검증 로직을 넣을 수 있다. 만약 추가적인 검증로직을 넣을 필요 없이 아이디와 패스워드만으로 검증한다면 스프링 시큐리티가 제공하는 AuthenticationProvider를 사용해도 충분하다.
* 만약 AuthenticationProvider를 구현하여 등록했다면 일반적으로 AuthenticationProvider에서 UserDetailsService를 주입받아 사용하기 때문에 auth.userDetailsService()를 통해 UserDetailsService를 등록할 필요가 없다.
* auth.authenticationProvider()를 통해 커스텀 AuthenticationProvider를 등록할 수 있다.
## WebAuthenticationDetails, AuthenticationDetailsSource(authentication 토큰에 추가적인 정보 삽입 가능)
```java
/*
authentication 의 멤버인 details(WebAuthenticationDetails 타입)에는 아이디와 비밀번호 외의 부가적인 정보가 들어간다.
클라이언트의 sessionId, address 가 기본적으로 들어가고 직접 WebAuthenticationDetails 를 상속한 클래스를 만들어서
다른 정보를 더 넣어도 된다. 만약 다른 정보를 넣을 필요가 없으면 굳이 아래와 같은 클래스를 만들 필요가 없다.
 */
@Getter
public class FormWebAuthenticationDetails extends WebAuthenticationDetails {

    private String secretKey;

    /**
     * Records the remote address and will also set the session Id if a session already
     * exists (it won't create one).
     *
     * @param request that the authentication request was received from
     */
    public FormWebAuthenticationDetails(HttpServletRequest request) {
        super(request);
        //secret_key라는 파라미터를 details에 추가
        secretKey = request.getParameter("secret_key");
    }
}
```
```java
//FormWebAuthenticationDetails 를 생성하는 클래스
//<context, return> -> context : buildDetails 메서드의 파라미터로 들어갈 타입, return : 반환 타입
@Component
public class FormWebAuthenticationDetailsSource implements AuthenticationDetailsSource<HttpServletRequest, WebAuthenticationDetails> {

    @Override
    public WebAuthenticationDetails buildDetails(HttpServletRequest request) {
        return new FormWebAuthenticationDetails(request);
    }
}
```
```java
@Configuration
@EnableWebSecurity
@RequiredArgsConstructor
public class SecurityConfig extends WebSecurityConfigurerAdapter {

    private final AuthenticationDetailsSource authenticationDetailsSource;

    @Override
    protected void configure(HttpSecurity http) throws Exception {
        http.authorizeRequests()
                .anyRequest().authenticated()

                .and().formLogin()
                .authenticationDetailsSource(authenticationDetailsSource) //직접 만든 AuthenticationDetailsSource 등록
    }

    ...

}
```
* WebAuthenticationDetails를 구현하여 SecurityContext에 있는 authentication에 부가적인 정보를 더 저장할 수 있다.
* WebAuthenticationDetails를 구현한 클래스를 사용하기 위해서는 AuthenticationDetailsSource를 구현해야한다. 이 때 오버라이드 하는 buildDetails 메서드는 내가 WebAuthenticationDetails를 구현해 만든 객체를 반환해야한다.
* http.authenticationDetailsSource()를 통해 내가 구현한 AuthenticationDetailsSource 객체를 등록할 수 있다.
## 커스텀 AuthenticationSuccessHandler(인증 성공시 수행할 로직)
```java
@Component
public class CustomAuthenticationHandler extends SimpleUrlAuthenticationSuccessHandler {

    //session 에 savedRequest 를 저장하고 가져오는 역할을 하는 객체
    private RequestCache requestCache = new HttpSessionRequestCache();

    //response.sendRedirect()를 통해 리다이렉트 할 수도 있겠지만 이거를 사용하는 편이 더 좋다.
    private RedirectStrategy redirectStrategy = new DefaultRedirectStrategy();

    @Override
    public void onAuthenticationSuccess(HttpServletRequest request, HttpServletResponse response, Authentication authentication) throws IOException, ServletException {

        //SimpleUrlAuthenticationHandler 가 상속하는 AbstractAuthenticationTargetUrlRequestHandler 의 메서드임
        setDefaultTargetUrl("/");

        SavedRequest savedRequest = requestCache.getRequest(request, response);
        if (savedRequest != null) {
            String targetUrl = savedRequest.getRedirectUrl();
            redirectStrategy.sendRedirect(request, response, targetUrl);
        } else {
            //getDefaultTargetUrl() 은 SimpleUrlAuthenticationHandler 가 상속하는 AbstractAuthenticationTargetUrlRequestHandler 의 메서드임
            redirectStrategy.sendRedirect(request, response, getDefaultTargetUrl());
        }
    }
}
```
```java
@Configuration
@EnableWebSecurity
@RequiredArgsConstructor
public class SecurityConfig extends WebSecurityConfigurerAdapter {

    private final AuthenticationSuccessHandler authenticationSuccessHandler;

    @Override
    protected void configure(HttpSecurity http) throws Exception {
        http.authorizeRequests()
                .anyRequest().authenticated()

                .and().formLogin()
                .successHandler(authenticationSuccessHandler)
    }

    ...

}
```
* AuthenticationSuccessHandler를 구현하여 로그인 성공시 수행할 로직을 구현할 수 있다.(SimpleUrlAuthenticationSuccessHandler는 AuthenticationSuccessHandler의 구현체)
* 로그인 성공시 로직은 `onAuthenticationSuccess(HttpServletRequest request, HttpServletResponse response, Authentication authentication)` 메서드를 오버라이드 해야한다. `onAuthenticationSuccess(HttpServletRequest request, HttpServletResponse response, FilterChain chain, Authentication authentication)` 를 사용하면 마지막에 chain.doFilter(request, response)를 넣어야하는데 그냥 `onAuthenticationSuccess(HttpServletRequest request, HttpServletResponse response, Authentication authentication)` 쓰는게 낫다.
## 커스텀 AuthenticationFailureHandler(인증 실패시 수행할 로직)
```java
@Component
public class CustomAuthenticationFailureHandler extends SimpleUrlAuthenticationFailureHandler {

    @Override
    public void onAuthenticationFailure(HttpServletRequest request, HttpServletResponse response, AuthenticationException exception) throws IOException, ServletException {
        //AuthenticationException 은 AuthenticationProvider 에서 인증이 실패했을 때 throw 된 exception 이다.

        String errorMessage = "Invalid Username or Password";

        if (exception instanceof InsufficientAuthenticationException) {
            errorMessage = "Invalid Secret Key"; //히든필드로 넘긴 secretKey 값이 "secret_key"가 아닐 때
        } else if (exception instanceof DisabledException) {
            errorMessage = "Locked";
        } else if (exception instanceof CredentialsExpiredException) {
            errorMessage = "Expired Password";
        }

        setDefaultFailureUrl("/login?error=true&exception=" + errorMessage);

        //SimpleUrlAuthentication 의 onAuthenticationFailure() 메서드에 위임
        super.onAuthenticationFailure(request, response, exception);
    }
}
```
```java
@Configuration
@EnableWebSecurity
@RequiredArgsConstructor
public class SecurityConfig extends WebSecurityConfigurerAdapter {

    private final AuthenticationFailureHandler authenticationFailureHandler;

    @Override
    protected void configure(HttpSecurity http) throws Exception {
        http.authorizeRequests()
                .anyRequest().authenticated()

                .and().formLogin()
                .failureHandler(authenticationFailureHandler)
    }

    ...

}
```
* AuthenticationFailureHandler를 구현하여 로그인 성공시 수행할 로직을 구현할 수 있다.(SimpleUrlAuthenticationFailureHandler는 AuthenticationFailureHandler의 구현체)
* 로그인 실패시 로직은 `onAuthenticationFailure(HttpServletRequest request, HttpServletResponse response, AuthenticationException exception)` 메서드를 오버라이드 하면 된다. 이 때 AuthenticationException은 AuthenticationProvider 에서 인증이 실패했을 때 throw 된 exception 이다.
## 커스텀 AccessDeniedHandler(인가 예외시 수행할 로직)
```java
@Setter
public class CustomAccessDeniedHandler implements AccessDeniedHandler {

    private String errorPage;

    //권한이 없는 리소스에 접근했을 때(인가 예외가 발생했을 때) 수행할 로직
    @Override
    public void handle(HttpServletRequest request, HttpServletResponse response, AccessDeniedException accessDeniedException) throws IOException, ServletException {
        //AccessDeniedException 은 인가 예외

        String deniedUrl = errorPage + "?exception=" + accessDeniedException.getMessage();
        response.sendRedirect(deniedUrl);
    }
}
```
```java
@Configuration
@EnableWebSecurity
@RequiredArgsConstructor
public class SecurityConfig extends WebSecurityConfigurerAdapter {

    @Bean
    public AccessDeniedHandler accessDeniedHandler() {
        CustomAccessDeniedHandler accessDeniedHandler = new CustomAccessDeniedHandler();
        accessDeniedHandler.setErrorPage("/denied");
        return accessDeniedHandler;
    }

    @Override
    protected void configure(HttpSecurity http) throws Exception {
        http.authorizeRequests()
                .anyRequest().authenticated();

        http.exceptionHandling()
                .accessDeniedHandler(accessDeniedHandler());
    }

    ...

}
```
* AccessDeniedHandler를 구현하여 인가 예외 발생시 사용되는 핸들러를 만들 수 있다.