# 커스텀 AccessDecisionVoter 추가하기
```java
//제네릭에는 요청 자원 정보가 들어있는 클래스가 들어간다.
//모든 요청에 대해 투표하길 원하면 Object 를 넣으면 된다.
@RequiredArgsConstructor
public class IpAddressVoter implements AccessDecisionVoter<Object> {

    private final SecurityResourceService securityResourceService;

    //해당 voter 가 ConfigAttribute(요청 자원에 대한 권한 정보들)를 보고 투표할 수 있는지 판단함
    @Override
    public boolean supports(ConfigAttribute attribute) {
        return true;
    }

    //해당 클래스타입(요청 자원 정보가 들어있는 객체의 타입)을 보고 투표할 수 있는지 판단함
    //보통 파라미터에는 FilterInvocation.class(요청 자원 정보)가 들어온다.
    @Override
    public boolean supports(Class<?> clazz) {
        return true;
    }

    //실질적인 투표를 하는 로직이 들어간다.
    @Override
    public int vote(Authentication authentication, Object object, Collection<ConfigAttribute> attributes) {

        WebAuthenticationDetails details = (WebAuthenticationDetails) authentication.getDetails();
        String remoteAddress = details.getRemoteAddress();

        List<String> accessIpList = securityResourceService.getAccessIpList();

        for (String s : accessIpList) {
            if (s.equals(remoteAddress)) {
                return ACCESS_ABSTAIN;
            }
        }

        throw new AccessDeniedException("인증되지 않은 IP 주소입니다.");
    }
}
```
```java
@Configuration
@EnableWebSecurity
@RequiredArgsConstructor
@Order(1)
public class SecurityConfig extends WebSecurityConfigurerAdapter {

    ...

    @Override
    public AuthenticationManager authenticationManagerBean() throws Exception {
        return super.authenticationManagerBean();
    }

    private List<AccessDecisionVoter<?>> getAccessDecisionVoters() {

        List<AccessDecisionVoter<? extends Object>> accessDecisionVoters = new ArrayList<>();
        //voter 를 추가할 때의 순서에 따라 결과가 달리질 수 있기 때문에 voter 의 추가 순서도 잘 생각해서 정해야한다.
        accessDecisionVoters.add(new IpAddressVoter(securityResourceService));
        accessDecisionVoters.add(new RoleVoter());
        return accessDecisionVoters;
    }

    @Bean
    public UrlFilterInvocationSecurityMetadataSource urlFilterInvocationSecurityMetadataSource() {
        return new UrlFilterInvocationSecurityMetadataSource(new LinkedHashMap<>());
    }

    @Bean
    public AccessDecisionManager accessDecisionManager() {
        return new AffirmativeBased(getAccessDecisionVoters());
    }

    @Bean
    public FilterSecurityInterceptor customFilterSecurityInterceptor() throws Exception {
        FilterSecurityInterceptor filterSecurityInterceptor = new FilterSecurityInterceptor();
        filterSecurityInterceptor.setSecurityMetadataSource(urlFilterInvocationSecurityMetadataSource());
        filterSecurityInterceptor.setAccessDecisionManager(accessDecisionManager());
        filterSecurityInterceptor.setAuthenticationManager(authenticationManagerBean());
        return filterSecurityInterceptor;
    }

    @Override
    protected void configure(HttpSecurity http) throws Exception {
        
        ...

        http.addFilterBefore(customFilterSecurityInterceptor(), FilterSecurityInterceptor.class);
    }

    ...

}
```
* AccessDecisionVoter의 투표하는 로직을 짤 때는 AccessDecisionManager의 구현체가 AffirmativeBased인지, ConsensusBased인지, UnanimousBased인지를 고려하고 로직을 짜야한다. 왜냐하면 각 구현체들이 다른 방식으로 AccessDecisionVoter의 투표결과를 가지고 요청을 grant 할지 deny 할지 판단하기 때문이다.
* AccessDecisionManager에 AccessDecisionVoter를 추가하는 순서에 따라 다른 결과가 나올 수 있기 때문에 AccessDecisionVoter를 추가할 때도 추가하는 순서를 잘 생각하고 정해야한다.