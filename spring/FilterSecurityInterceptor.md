# 커스텀 SecurityMetadataSource 등록 및 FilterSecurityInterceptor 등록
```java
public class UrlFilterInvocationSecurityMetadataSource implements FilterInvocationSecurityMetadataSource {

    private final Map<RequestMatcher, Collection<ConfigAttribute>> requestMap;

    public UrlFilterInvocationSecurityMetadataSource(
            LinkedHashMap<RequestMatcher, Collection<ConfigAttribute>> requestMap) {
        this.requestMap = requestMap;
    }

    //MetadataSource 에 Map 의 형태로 저장되어 있는 자원에 대한 권한 정보를 가져오는 메서드이다.
    @Override
    public Collection<ConfigAttribute> getAttributes(Object object) throws IllegalArgumentException {

        /*
        url 방식의 MetadataSource 면 FilterInvocation 이 들어오지만 메서드 방식이면 MethodInvocation 이 들어오기 때문에
        Object 타입을 받고 그걸 타입 캐스팅 해서 써야한다.
         */
        HttpServletRequest request = ((FilterInvocation) object).getRequest();

        if (requestMap != null) {
            for (Map.Entry<RequestMatcher, Collection<ConfigAttribute>> entry : requestMap.entrySet()) {
                if (entry.getKey().matches(request)) {
                    return entry.getValue();
                }
            }
        }

        return null;
    }

    @Override
    public Collection<ConfigAttribute> getAllConfigAttributes() {
        //모든 권한 정보를 가져오는 메서드이다.
        Set<ConfigAttribute> allAttributes = new HashSet<>();
        this.requestMap.values().forEach(allAttributes::addAll);
        return allAttributes;
    }

    @Override
    public boolean supports(Class<?> clazz) {
        //url 방식으로 권한을 부여하면 getAttributes 의 파라미터로 FilterInvocation, 메서드 방식이면 MethodInvocation 을 받는다.
        return FilterInvocation.class.isAssignableFrom(clazz);
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
        return List.of(new RoleVoter());
    }

    @Bean
    public UrlFilterInvocationSecurityMetadataSource urlFilterInvocationSecurityMetadataSource() {
        return new UrlFilterInvocationSecurityMetadataSource(new LinkedHashMap<>());
    }

    @Bean
    public AccessDecisionManager accessDecisionManager() {
        //AccessDecisionVoter 는 AccessDecisionManager 를 스프링 빈으로 등록할 때 주입해야 한다.
        return new AffirmativeBased(getAccessDecisionVoters());
    }

    /*
    MetadataSource, AccessDecisionManager, AccessDecisionVoter 등을 설정하려면 FilterSecurityInterceptor의 setter를 사용해 설정한 후 해당 FilterSecurityInterceptor를 addFilterBefore를 통해 기존의 FilterSecurityInterceptor 앞에 추가해줘야 한다.
     */
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

        /*
        기본으로 등록되는 FilterSecurityInterceptor 앞에 내가 생성한 FilterSecurityInterceptor 를 등록해야
        내가 생성한 FilterSecurityInterceptor 가 동작한다. 한번 FilterSecurityInterceptor 에 의해
        권한 검사가 진행되면 다음 필터로 넘어가더라도 권한 검사를 진행하지 않는다.(그렇게 로직이 짜여있음)
         */
        http.addFilterBefore(customFilterSecurityInterceptor(), FilterSecurityInterceptor.class);
    }

    ...

}
```
* AccessDecisionManager의 구현체를 바꾸거나, AccessDecisionVoter를 추가하거나, 직접 만든 SecurityMetadataSource를 사용해 자원에 대한 권한을 검사하고 싶을 때 FilterSecurityInterceptor를 addFilterBefore를 사용해 추가해 줘야한다.
* 만약 기존의 FilterSecurityInterceptor 앞에 직접 FilterSecurityInterceptor를 추가해 줬다면 해당 FilterSecurityInterceptor가 동작한 후 request의 "__spring_security_filterSecurityInterceptor_filterApplied" 속성이 true로 세팅되어 기존의 FilterSecurityInterceptor는 동작하지 않는다.