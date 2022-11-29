# configure(WebSecurity web)
## deprecated 전
```java
@EnableWebSecurity
public class SecurityConfig extends WebSecurityConfigurerAdapter {
 
    @Override
    public void configure(WebSecurity web) throws Exception {
        //webIgnore 설정, 정적 리소스들이 스프링 시큐리티 필터를 거치지 않는다.
        web.ignoring().requestMatchers(PathRequest.toStaticResources().atCommonLocations());
    }

    ...

}
```
## deprecated 후
```java
@EnableWebSecurity
public class SecurityConfig {
    
    @Bean
    public WebSecurityCustomizer webSecurityCustomizer() {
        return web -> web.ignoring().requestMatchers(PathRequest.toStaticResources().atCommonLocations());
    }

    ...

}
```
# configure(HttpSecurity http)
## deprecated 전
```java
@EnableWebSecurity
public class SecurityConfig extends WebSecurityConfigurerAdapter {
    
    @Override
    protected void configure(HttpSecurity http) throws Exception {
        
        http.authorizeRequests()
                .anyRequest().authenticated();
        http.formLogin();
    }

    ...

}
```
## deprecated 후
```java
@EnableWebSecurity
public class SecurityConfig {
    
    @Bean
    public SecurityFilterChain securityFilterChain(HttpSecurity http) throws Exception {
        http.authorizeRequests()
                .anyRequest().authenticated();
        http.formLogin();
        return http.build();
    }

    ...

}
```
# configure(AuthenticationManagerBuilder auth)를 통한 UserDetailsService, AuthenticationProvider 등 등록
## deprecated 전
```java
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
        
        auth.userDetailsService(userDetailsService);
        auth.authenticationProvider(customAuthenticationProvider());
    }

    ...

}
```
* deprecated 이전에는 AuthenticationManagerBuilder를 통해 UserDetailsService나 AuthenticationProvider 등을 등록해야한다.
## deprecated 후
```java
@EnableWebSecurity
public class SecurityConfig {
    
    @Bean
    UserDetailsService userDetailsService(UserRepository userRepository) {
        return new CustomUserDetailsService(userRepository);
    }

    @Bean
    public AuthenticationProvider authenticationProvider(UserDetailsService userDetailsService, PasswordEncoder passwordEncoder) {
        return new CustomAuthenticationProvider(userDetailsService, passwordEncoder);
    }

    ...
    
}
```
* deprecated 후에는 스프링 빈으로 만들어주면 자동으로 해당 UserDetailsService와 AuthenticationProvider 등을 사용한다.