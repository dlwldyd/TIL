# FactoryBean
```java
@RequiredArgsConstructor
@Component
public class UrlResourcesMapFactoryBean implements FactoryBean<LinkedHashMap<RequestMatcher, List<ConfigAttribute>>> {

    private final SecurityResourceService securityResourceService;
    
    private LinkedHashMap<RequestMatcher, List<ConfigAttribute>> resourceMap;

    //해당 타입의 객체를 생성해(싱글톤 아닐 때) 반환함
    @Override
    public LinkedHashMap<RequestMatcher, List<ConfigAttribute>> getObject() throws Exception {

        if (resourceMap == null) {
            init();
        }

        return resourceMap;
    }

    private void init() {
        resourceMap = securityResourceService.getResourceList();
    }

    //어떤 타입의 객체를 생성하는지 반환함
    @Override
    public Class<?> getObjectType() {
        return LinkedHashMap.class;
    }

    //생성되는 객체가 싱글톤인지 아닌지
    @Override
    public boolean isSingleton() {
        return true;
    }
}
```
* FactoryBean<반환하는 객체 타입> 을 구현해 사용할 수 있다.
* 단순 new로 생성하기 힘든 객체들을 빈으로 활용할 수 있게끔 하기 위한 클래스이다. 생성이 힘든 객체를 FactoryBean의 getObject() 메서드에서 생성하는 로직을 짜고 FactoryBean을 스프링 빈으로 등록함으로써 스프링은 getObject()메서드가 반환하는 객체를 스프링 빈으로 등록한다.