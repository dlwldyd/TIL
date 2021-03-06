# 커스텀 DSL 구현(필터별, 도메인별 보안 설정 분리)
```java
//스프링 시큐리티 설정 API 로 사용 가능
//SecurityConfigurer 를 구현해서 만들 수 있음, AbstractAuthenticationFilterConfigurer 는 SecurityConfigurer 의 구현체
//SecurityConfigurer 를 구현하기에는 기능도 적고 기능을 전부 구현하려면 시간이 많이 드니깐 AbstractAuthenticationFilterConfigurer 를 상속받자
/*
<H extends HttpSecurityBuilder<H>> -> 어떤 configure 메서드에서 설정할지
configure(HttpSecurity http)
configure(AuthenticationManagerBuilder auth)
configure(WebSecurity web)
 */
//<H, AjaxLoginConfigurer<H>, AjaxLoginProcessingFilter> -> <H, AjaxLoginConfigurer<H>, 적용할 필터>
public final class AjaxLoginConfigurer<H extends HttpSecurityBuilder<H>> extends
        AbstractAuthenticationFilterConfigurer<H, AjaxLoginConfigurer<H>, AjaxLoginProcessingFilter> {

    private AuthenticationSuccessHandler successHandler;
    private AuthenticationFailureHandler failureHandler;
    private AuthenticationManager authenticationManager;

    public AjaxLoginConfigurer() {
        //필터와 해당 필터가 적용될 url
        super(new AjaxLoginProcessingFilter(), "/api/login");
    }

    @Override
    public void init(H http) throws Exception {
        super.init(http);
    }

    //WebSecurityConfigurerAdapter 의 configure 메서드가 실행된 후 실행됨
    @Override
    public void configure(H http) {
        //apply 시점에 적용됨

        if(authenticationManager == null){
            //스프링 시큐리티에서 공유하는 객체를 가져오는 메서드
            //Map<Class<?>, Object> 에서 꺼내옴
            authenticationManager = http.getSharedObject(AuthenticationManager.class);
        }
        getAuthenticationFilter().setAuthenticationManager(authenticationManager);
        getAuthenticationFilter().setAuthenticationSuccessHandler(successHandler);
        getAuthenticationFilter().setAuthenticationFailureHandler(failureHandler);

        SessionAuthenticationStrategy sessionAuthenticationStrategy = http
                .getSharedObject(SessionAuthenticationStrategy.class);
        if (sessionAuthenticationStrategy != null) {
            getAuthenticationFilter().setSessionAuthenticationStrategy(sessionAuthenticationStrategy);
        }
        RememberMeServices rememberMeServices = http
                .getSharedObject(RememberMeServices.class);
        if (rememberMeServices != null) {
            //내가 설정한 인증 필터(여기서는 AjaxLoginProcessingFilter)의 RememberMeServices 를 설정
            getAuthenticationFilter().setRememberMeServices(rememberMeServices);
        }
        //Map<Class<?>, Object> 에 내가 설정한 인증 필터(여기서는 AjaxLoginProcessingFilter) 넣음
        http.setSharedObject(AjaxLoginProcessingFilter.class, getAuthenticationFilter());
        http.addFilterBefore(getAuthenticationFilter(), UsernamePasswordAuthenticationFilter.class);
    }

    public AjaxLoginConfigurer<H> successHandlerAjax(AuthenticationSuccessHandler successHandler) {
        this.successHandler = successHandler;
        return this;
    }

    public AjaxLoginConfigurer<H> failureHandlerAjax(AuthenticationFailureHandler authenticationFailureHandler) {
        this.failureHandler = authenticationFailureHandler;
        return this;
    }

    public AjaxLoginConfigurer<H> setAuthenticationManager(AuthenticationManager authenticationManager) {
        this.authenticationManager = authenticationManager;
        return this;
    }

    @Override
    protected RequestMatcher createLoginProcessingUrlMatcher(String loginProcessingUrl) {
        return new AntPathRequestMatcher(loginProcessingUrl, "POST");
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

    @Bean
    public AuthenticationSuccessHandler ajaxAuthenticationSuccessHandler() {
        return new AjaxAuthenticationSuccessHandler();
    }

    @Bean
    public AuthenticationFailureHandler ajaxAuthenticationFailureHandler() {
        return new AjaxAuthenticationFailureHandler();
    }

    @Bean
    public AccessDeniedHandler ajaxAccessDeniedHandler() {
        return new AjaxAccessDeniedHandler();
    }

    @Override
    public AuthenticationManager authenticationManagerBean() throws Exception {
        return super.authenticationManagerBean();
    }

    @Override
    protected void configure(AuthenticationManagerBuilder auth) throws Exception {
        auth.authenticationProvider(ajaxAuthenticationProvider(userDetailsService, passwordEncoder));
    }

    @Override
    protected void configure(HttpSecurity http) throws Exception {

        http
                .antMatcher("/api/**")
                .authorizeRequests()
                .antMatchers("/api/messages").hasRole("MANAGER")
                .anyRequest().authenticated();

        //커스텀 DSL을 만들어 일반적인 스프링 시큐리티 API를 통한 설정처럼 사용할 수 있다.
        http.apply(new AjaxLoginConfigurer<>())
                .successHandlerAjax(ajaxAuthenticationSuccessHandler())
                .failureHandlerAjax(ajaxAuthenticationFailureHandler())
                .setAuthenticationManager(authenticationManagerBean());

        http.exceptionHandling()
                .authenticationEntryPoint(new AjaxLoginAuthenticationEntryPoint())
                .accessDeniedHandler(ajaxAccessDeniedHandler());

        http.csrf()
                .disable();
    }
}
```
* 필터별 도메인별로 보안 설정을 분리하여 더 깔끔하게 보안설정을 할 수 있다. 하지만 보안시스템 규모가 그렇게 크지 않으면 DSL로 구조화 하는 것 보다 그냥 http 방식을 사용하는 것이 오히려 더 편한 것 같다.
* SecurityConfigurer를 구현하여 커스텀 DSL을 만들 수 있다. 다만 SecurityConfigurer를 구현하기에는 기능도 적고 기능을 전부 구현하려면 시간이 많이 드니깐 AbstractAuthenticationFilterConfigurer를 상속받자