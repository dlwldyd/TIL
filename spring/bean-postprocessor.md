# 빈 후처리기
<center><img src="./../img/bean-postprocessor.jpg"></center>

```java
public class PackageLogTracePostProcessor implements BeanPostProcessor {

    private final String basePackage;
    private final Advisor advisor;

    public PackageLogTracePostProcessor(String basePackage, Advisor advisor) {
        this.basePackage = basePackage;
        this.advisor = advisor;
    }

    //@PostContruct 가 붙은 메서드 적용 이후
    @Override
    public Object postProcessAfterInitialization(Object bean, String beanName) throws BeansException {
        //프록시 적용 대상여부 체크
        //프록시 적용 대상이 아니면 원본을 그대로 넘김
        String packageName = bean.getClass().getPackageName();
        if (!packageName.startsWith(basePackage)) {
            return bean;
        }

        //프록시 대상이면 프록시를 만들어서 반환
        ProxyFactory proxyFactory = new ProxyFactory(bean);
        proxyFactory.addAdvisor(advisor);
        Object proxy = proxyFactory.getProxy();
        return proxy;
    }

    /*@PostConstruct 가 붙은 메서드 적용 이전
    @Override
    public Object postProcessBeforeInitialization(Object bean, String beanName) throws BeansException {
        return BeanPostProcessor.super.postProcessBeforeInitialization(bean, beanName);
    }
    */
}
```
```java
@Configuration
public class BeanPostProcessorConfig {

    @Bean
    public PackageLogTracePostProcessor logTracePostProcessor() {
        return new PackageLogTracePostProcessor("com.example.proxy.app", getAdvisor());
    }

    private Advisor getAdvisor() {

        NameMatchMethodPointcut pointcut = new NameMatchMethodPointcut();
        pointcut.setMappedNames("request", "orderItem", "save");

        LogTraceAdvice advice = new LogTraceAdvice();
        return new DefaultPointcutAdvisor(pointcut, advice);
    }
}
```
```java
@Slf4j
public class LogTraceAdvice implements MethodInterceptor {

    @Override
    public Object invoke(MethodInvocation invocation) throws Throwable {
        try {
            Method method = invocation.getMethod();
            String message = method.getDeclaringClass().getSimpleName() + "." + method.getName() + "()";
            log.info("{} 프록시 적용", message);

            Object result = invocation.proceed();

            return result;
        } catch (Exception e) {
            log.info("예외 발생");
            throw e;
        }
    }
}
```
* 빈 후처리기는 BeanPostProcessor를 구현하여 만들 수 있다. 그리고 빈 후처리기를 사용하려면 빈 후처리기를 스프링 빈으로 등록하여야 한다.
* 빈 후처리기는 빈을 조작하고 변경할 수 있는 후킹 포인트이다.
이것은 빈 객체를 조작하거나 다른 객체로 바꾸어 버릴 수 있다.
* 빈 후처리기를 사용하면 컴포넌트 스캔으로 등록한 빈 객체에 대한 프록시를 생성할 수 있다. 또한 각각의 빈 객체에 대한 프록시 생성 코드를 빈 후처리기 한 곳에서 관리하여 코드량을 줄일 수 있다.
* postProcessAfterInitialization : 빈 객체 초기화 이후 호출된다.(@PostContruct가 붙은 메서드 호출 이후)
* postProcessBeforeInitialization : 빈 객체 초기화 이전에 호출된다.(@PostContruct가 붙은 메서드 호출 이전)