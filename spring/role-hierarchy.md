# 계층 권한 적용하기
```java
@Configuration
@EnableWebSecurity
@RequiredArgsConstructor
@Order(1)
public class SecurityConfig extends WebSecurityConfigurerAdapter {

    ...

    @Bean
    public AccessDecisionManager accessDecisionManager() {
        return new AffirmativeBased(getAccessDecisionVoters());
    }

    private List<AccessDecisionVoter<?>> getAccessDecisionVoters() {

        List<AccessDecisionVoter<? extends Object>> accessDecisionVoters = new ArrayList<>();
        accessDecisionVoters.add(roleVoter());
        return accessDecisionVoters;
    }

    @Bean
    public AccessDecisionVoter<? extends Object> roleVoter() {
        return new RoleHierarchyVoter(roleHierarchy());
    }

    @Bean
    public RoleHierarchyImpl roleHierarchy() {
        return new RoleHierarchyImpl();
    }

    roleHierarchy().setHierarchy("ROLE_ADMIN > ROLE_MANAGER\nROLE_MANAGER > ROLE_USER\n");

    @Bean
    public FilterSecurityInterceptor customFilterSecurityInterceptor() throws Exception {
        FilterSecurityInterceptor filterSecurityInterceptor = new FilterSecurityInterceptor();
        filterSecurityInterceptor.setSecurityMetadataSource(urlFilterInvocationSecurityMetadataSource());
        filterSecurityInterceptor.setAccessDecisionManager(accessDecisionManager());
        filterSecurityInterceptor.setAuthenticationManager(authenticationManagerBean());
        return filterSecurityInterceptor;
    }

    ...

}
```
* 권한 계층을 구현하기 위해서는 RoleHierarchyVoter를 voter로 추가해야한다.
* RoleHierarchyVoter를 생성할 때 생성자의 매개변수로 RoleHierarchy를 받는다. RoleHierarchyImpl은 RoleHierarchy의 구현체인데 일반적으로는 RoleHierarchyImpl을 사용한다.
* RoleHierarchyImpl의 setHierarchy 메서드로 권한계층을 적용할 수 있는데 메서드의 매개변수로 들어가는 문자열은 `권한 > 권한\n`과 같은 문자열의 연속이다.(비교연산자와 권한 사이에 띄워줘야함)