# 다중 보안 설정
<center><img src="./../img/multi-config.png"></center>

```java
@Configuration
@EnableWebSecurity
@Order(0) //Order 는 구체적인 경로에 매치되는 SecurityConfig 가 더 앞에 오도록 해야함
public class SecurityConfig2 extends WebSecurityConfigurerAdapter {

    @Override
    protected void configure(HttpSecurity http) throws Exception {
        http.antMatcher("/admin/config2")
                .authorizeRequests()
                .anyRequest().permitAll()
                .and()
                .httpBasic();
    }
}
```
```java
@Configuration
@EnableWebSecurity
@RequiredArgsConstructor
@Order(1)
public class SecurityConfig1 extends WebSecurityConfigurerAdapter {
    
    @Override
    protected void configure(HttpSecurity http) throws Exception {
     
        http.authorizeRequests()
            .antMatchers("/denied").permitAll()
            .antMatchers("/user").hasRole("USER")
            .antMatchers("/admin/pay").hasRole("ADMIN")
            .antMatchers("/admin/**").access("hasRole('ADMIN') or hasRole('SYS')")
            .anyRequest().authenticated()
            .and()
            .formLogin();
    }
}
```
* 여러개의 SecurityConfig 클래스를 만들어 클라이언트의 요청 url에 따라 서로 다른 보안을 적용할 수 있다.
* 여러개의 SecurityConfig 클래스를 만들어 적용하려면 @Order를 통해 클래스의 순서를 지정해 줘야한다. 각각의 SecurityConfig 클래스로 인해 만들어지는 SecurityFilterChain 객체는 SecurityFilterChains 라는 List에 담겨져 앞부분 부터 순서대로 요청의 url과 RequestMatcher를 비교하고 만약 매칭이 참이되면 뒤쪽의 RequestMatcher와는 비교를 하지 않기 때문에 구체적인 url경로를 가진 SecurityConfig 클래스의 순서가 더 앞에 있어야 한다.
* 동작 순서
  1. 보안 설정으로 인해 만들어진 각각의 필터들과 RequestMatcher들이 SecurityFilterChain이라는 객체로 만들어진다.
  2. FilterChainProxy 객체의 SecurityFilterChains라는 List에 만들어진 SecurityFilterChain 객체가 담긴다. 이때 @Order()로 지정된 순서대로 List에 담긴다.
  3. 클라이언트의 요청이 들어온 경우 FilterChainProxy는 클라이언트가 요청한 url을 SecurityFilterChains(List)의 앞 순서부터 순서대로 RequestMatcher와 매칭하여 만약 참이면 SecurityFilterChain에 담긴 Filters에 request를 전달하여 보안처리를 한다.