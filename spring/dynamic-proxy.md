# 동적 프록시
## 리플렉션
```java
public class Hello {
    public String callA() {
        return "A";
    }

    public String callB() {
        return "B";
    }
}
```
```java
@Test
public void reflection() throws Exception {
    //클래스 메타 정보 얻기, 내부 클래스를 사용하는 경우는 $를 써야한다.
    Class classHello = Class.forName("com.example.dynamicproxy.Reflection.Hello");

    Hello target = new Hello();
    //callA 메서드 정보
    Method methodCallA = classHello.getMethod("callA");
    dynamicCall(methodCallA, target);

    //callB 메서드 정보
    Method methodCallB = classHello.getMethod("callB");
    dynamicCall(methodCallB, target);
}
```
```java
private void dynamicCall(Method method, Object target) throws Exception {
    //공통 로직 시작
    log.info("start");
    Object result = method.invoke(target);
    log.info("result={}", result);
    //공통 로직 종료
}
```
* 만약 callA()와 callB()를 그대로 호출한다면 log.info("start")와 log.info("result={}", result)를 두번씩 코딩해야 한다. 하지만 Reflection을 이용한다면 클래스나 메서드의 메타정보를 동적으로 획득하고 코드도 동적으로 호출할 수 있다. 그로인해 공통으로 사용되는 로직을 한번만 코딩할 수 있다.
* reflection은 호출할 메서드가 런타임에 결정되기 때문에 에러가 컴파일 시점이 아닌 런타임 시점에 발생한다. 따라서 reflection을 사용하는 것이 꼭 필요한 경우가 아니라면 지양하는게 좋다.
## JDK 동적 프록시
<center><img src="./../img/JDK-dynamic-proxy.png"></center>

```java
public interface AInterface {
    String call();
}
```
```java
public interface BInterface {
    String call();
}
```
```java
public class AImpl implements AInterface {

    @Override
    public String call() {
        return "a";
    }
}
```
```java
public class BImpl implements BInterface {

    @Override
    public String call() {
        return "b";
    }
}
```
```java
@Slf4j
public class TimeInvocationHandler implements InvocationHandler {

    //프록시가 호출할 대상(다음 프록시 혹은 실제 기능을 수행하는 객체)
    private final Object target;

    public TimeInvocationHandler(Object target) {
        this.target = target;
    }

    //프록시로 적용할 공통 로직이 들어간다.
    //proxy : 프록시 자신, method : 호출한 메서드, args : 메서드를 호출할 때 전달할 파라미터
    @Override
    public Object invoke(Object proxy, Method method, Object[] args) throws Throwable {
        log.info("TimeProxy 실행");
        long startTime = System.currentTimeMillis();

        Object result = method.invoke(target, args);

        long endTime = System.currentTimeMillis();
        long resultTime = endTime - startTime;
        log.info("TimeProxy 종료 resultTime={}", resultTime);
        return result;
    }
}
```
```java
public class JdkDynamicProxyTest {

    @Test
    void dynamicA() {
        AInterface target = new AImpl();
        TimeInvocationHandler handler = new TimeInvocationHandler(target);

        //동적 프록시 생성(java.lang.reflect.Proxy)
        AInterface proxy = (AInterface) Proxy.newProxyInstance(AInterface.class.getClassLoader(), new Class[]{AInterface.class}, handler);

        proxy.call();
    }

    @Test
    void dynamicB() {
        BInterface target = new BImpl();
        TimeInvocationHandler handler = new TimeInvocationHandler(target);

        BInterface proxy = (BInterface) Proxy.newProxyInstance(BInterface.class.getClassLoader(), new Class[]{BInterface.class}, handler);

        proxy.call();
    }
}
```
* JDK 동적 프록시에 적용할 로직은 InvocationHandler 인터페이스를 구현해서 작성하면 된다.
* JDK 동적 프록시는 인터페이스에 기반한 프록시만 생성할 수 있다. 구체클래스에 기반한 프록시를 생성하려면 CGLIB를 사용해야한다.([프록시 패턴 참고](../design%20pattern/proxy-pattern.md))
* JDK 동적 프록시를 사용하면 서로 다른 인터페이스를 상속받는 구체클래스에 대한 프록시를 인터페이스 개수만큼 만들지 않고 하나만 만들어도 프록시를 생성할 수 있다.
## CGLIB
```java
@Slf4j
public class ConcreteService {
    public void call() {
        log.info("ConcreteService 호출");
    }
}
```
```java
@Slf4j
public class TimeMethodInterceptor implements MethodInterceptor {

    private final Object target;

    public TimeMethodInterceptor(Object target) {
        this.target = target;
    }

    @Override
    public Object intercept(Object obj, Method method, Object[] args, MethodProxy methodProxy) throws Throwable {
        log.info("TimeProxy 실행");
        long startTime = System.currentTimeMillis();

        Object result = methodProxy.invoke(target, args);

        long endTime = System.currentTimeMillis();
        long resultTime = endTime - startTime;
        log.info("TimeProxy 종료 resultTime={}", resultTime);
        return result;
    }
}
```
```java
@Slf4j
public class CglibTest {

    @Test
    void cglib() {
        ConcreteService target = new ConcreteService();

        Enhancer enhancer = new Enhancer();
        enhancer.setSuperclass(ConcreteService.class);
        enhancer.setCallback(new TimeMethodInterceptor(target));
        ConcreteService proxy = (ConcreteService) enhancer.create();
        log.info("targetClass={}", target.getClass());
        log.info("proxyClass={}", proxy.getClass());

        proxy.call();
    }
}
```
* CGLIB을 이용해 프록시를 만들 때 적용할 로직은 MethodInterceptor(org.springframework.cglib.proxy.MethodInterceptor)를 구현해서 만든다.
* 구체클래스를 기반으로 하는 프록시를 만들 때는 JDK 동적프록시가 아닌 CGLIB으로 만들어야 한다. JDK 동적프록시는 인터페이스를 implements 해서 만들지만 CGLIB는 구체클래스를 extends해서 만들기 때문이다.
* 스프링 AOP에서는 default로 CGLIB를 통해 프록시를 생성하도록 하고 있다. JDK동적 프록시로 프록시를 생성하면 해당 프록시는 구체클래스로 타입캐스팅이 불가능하다는 단점이 있기 때문이다. CGLIB도 final이 붙은 클래스에대한 프록시를 만들지 못한다는 단점이 있지만 클래스나 메서드에 final을 붙이는 경우가 거의 없으므로 큰 단점이 되지 않는다.
## 프록시 팩토리
<center><img src="./../img/proxy-factory2.png"></center>

```java
public interface ServiceInterface {
    void save();
    void find();
}
```
```java
@Slf4j
public class ServiceImpl implements ServiceInterface{

    @Override
    public void save() {
        log.info("save 호출");
    }

    @Override
    public void find() {
        log.info("find 호출");
    }
}
```
```java
@Slf4j
public class TimeAdvice implements MethodInterceptor {

    @Override
    public Object invoke(MethodInvocation invocation) throws Throwable {
        log.info("TimeProxy 실행");
        long startTime = System.currentTimeMillis();

        Object result = invocation.proceed();

        long endTime = System.currentTimeMillis();
        long resultTime = endTime - startTime;
        log.info("TimeProxy 종료 resultTime={}", resultTime);
        return result;
    }
}
```
```java
@Slf4j
public class ProxyFactoryTest {
    @Test
    void proxyTargetClass() {
        ServiceInterface target = new ServiceImpl();
        ProxyFactory proxyFactory = new ProxyFactory(target);
        //setProxyTargetClass()를 true 로 세팅하면 타겟 클래스(ServiceImpl)를 상속하여 프록시 생성(항상 CGLIB 를 기반으로 프록시 생성)
        //proxyFactory.setProxyTargetClass(true);
        DefaultPointcutAdvisor advisor = new DefaultPointcutAdvisor(Pointcut.TRUE, new TimeAdvice());
        proxyFactory.addAdvisor(advisor);
        //proxyFactory.addAdvice(new TimeAdvice()); 위의 2라인과 같다.
        ServiceInterface proxy = (ServiceInterface) proxyFactory.getProxy();

        proxy.save();

        //프록시 팩토리를 통해서 프록시를 생성했을 때만 AopUtils 사용 가능
        assertThat(AopUtils.isAopProxy(proxy)).isTrue();
        assertThat(AopUtils.isJdkDynamicProxy(proxy)).isTrue();
        assertThat(AopUtils.isCglibProxy(proxy)).isFalse();
    }
}
```
* 프록시 팩토리를 사용하면 인터페이스를 기반으로 하는 프록시와 구체클래스를 기반으로 하는 프록시를 따로 만들어 관리할 필요 없다.
* 프록시 팩토리는 인터페이스가 있는 경우 JDK 동적 프록시를 적용해 프록시를 생성하고 그렇지 않은 경우에는 CGLIB를 적용해 프록시를 생성한다. 단, proxyFactory.setProxyTargetClass(true)를 하면 항상 CGLIB를 적용해서 프록시를 생성한다.
* MethodInterceptor(org.aopalliance.intercept.MethodInterceptor)를 구현해서 프록시에 적용할 로직을 작성하면 된다.
## 포인트컷, Advice, Advisor
<center><img src="./../img/proxy-factory.png"></center>

### 포인트 컷
```java
public class MyPointcut implements Pointcut {

    @Override
    public ClassFilter getClassFilter() {
        return ClassFilter.TRUE;
    }

    @Override
    public MethodMatcher getMethodMatcher() {
        return new MyMethodMatcher();
    }
}
```
```java
public class MyMethodMatcher implements MethodMatcher {

    private String matchName = "save";

    @Override
    public boolean matches(Method method, Class<?> targetClass) {
        boolean result = method.getName().equals(matchName);
        method.getName(), targetClass);
        return result;
    }

    @Override
    public boolean isRuntime() {
        return false;
    }

    @Override
    public boolean matches(Method method, Class<?> targetClass, Object... args) {
        return false;
    }
}
```
* 포인트컷은 어디에 부가 기능을 적용할지, 어디에 부가 기능을 적용하지 않을지 판단하는 필터링 로직이다. 주로 클래스와 메서드 이름으로 필터링 한다.
* 포인트컷은 Pointcut을 구현해서 만든다.
* 일반적으로 스프링에서 제공하는 포인트컷을 사용하고 직접 만들 일은 거의 없다.
* 포인트컷을 적용할 때 Pointcut.TRUE를 적용하면 항상 true가 반환되어서 모든 메서드에 어드바이스가 적용된다.
* 스프링이 제공하는 포인트컷 중 가장 중요하고 자주 쓰이는 포인트컷은 AspectJExpressionPointcut이다.
### Advice
```java
@Slf4j
public class TimeAdvice implements MethodInterceptor {

    @Override
    public Object invoke(MethodInvocation invocation) throws Throwable {
        log.info("TimeProxy 실행");
        long startTime = System.currentTimeMillis();

        Object result = invocation.proceed();

        long endTime = System.currentTimeMillis();
        long resultTime = endTime - startTime;
        log.info("TimeProxy 종료 resultTime={}", resultTime);
        return result;
    }
}
```
* 어드바이스는 프록시가 호출하는 부가 기능이다. 단순하게 프록시 로직이라 생각하면 된다.
* MethodInterceptor(org.aopalliance.intercept.MethodInterceptor)를 구현해서 어드바이스를 작성할 수 있다.
### Advisor
```java
public interface ServiceInterface {
    void save();
    void find();
}
```
```java
@Slf4j
public class ServiceImpl implements ServiceInterface{

    @Override
    public void save() {
        log.info("save 호출");
    }

    @Override
    public void find() {
        log.info("find 호출");
    }
}
```
```java
@Slf4j
public class Advice1 implements MethodInterceptor {

    @Override
    public Object invoke(MethodInvocation invocation) throws Throwable {
        log.info("Advice1 호출");
        return invocation.proceed();
    }
}
```
```java
@Slf4j
public class Advice2 implements MethodInterceptor {

    @Override
    public Object invoke(MethodInvocation invocation) throws Throwable {
        log.info("Advice2 호출");
        return invocation.proceed();
    }
}
```
```java
@Test
void AdvisorTest() {
    
    //client -> proxy -> advisor2 -> advisor1 -> target
    NameMatchMethodPointcut pointcut = new NameMatchMethodPointcut();
    pointcut.setMappedName("save");

    DefaultPointcutAdvisor advisor1 = new DefaultPointcutAdvisor(pointcut, new Advice1());
    DefaultPointcutAdvisor advisor2 = new DefaultPointcutAdvisor(pointcut, new Advice2());

    ServiceInterface target = new ServiceImpl();
    ProxyFactory proxyFactory = new ProxyFactory(target);

    proxyFactory.addAdvisor(advisor2);
    proxyFactory.addAdvisor(advisor1);
    ServiceInterface proxy = (ServiceInterface) proxyFactory.getProxy();

    proxy.save();
    proxy.find();
}
```
* 어드바이저는 포인트컷 하나와 어드바이스 하나를 가지고 있는 객체라 보면 된다.
* 어드바이저는 스프링 AOP에서만 사용되는 용어이다.
* proxyFactory.addAdvisor()를 통해 여러개의 어드바이저를 적용할 수 있다. 적용되는 순서는 어드바이저를 추가한 순서와 같다. 이 때 어드바이저의 개수만큼 프록시가 생성되는 것이 아니라 하나의 프록시만 생성되고 그 안에 어드바이스를 호출하는 메서드가 어드바이저의 개수만큼 있어서 차례대로 호출된다.
## AnnotationAwareAspectJAutoProxyCreator
```java
@Slf4j
@Configuration
public class AutoProxyConfig {

    @Bean
    public Advisor advisor() {

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
* implementation 'org.springframework.boot:spring-boot-starter-aop'를 build.gradle에 추가하면 스프링 부트가 AOP 관련 클래스를 자동으로 스프링 빈에 등록하는데 이 때 AnnotationAwareAspectJAutoProxyCreator라는 빈 후처리기([빈 후처리기 참고](bean-postprocessor.md))가 스프링 빈으로 자동 등록된다.
* AnnotationAwareAspectJAutoProxyCreator는 자동으로 프록시를 생성해 주는 빈 후처리기다. 개발자가 어드바이저를 스프링 빈으로 등록해 두면 이 빈 후처리기는 스프링 빈으로 등록된 모든 어드바이저를 찾아서 모든 스프링 빈을 검사해 어드바이저에 있는 포인트컷이 하나라도 참이 되면 프록시를 생성해 원본 대신 프록시를 등록한다. 다만 객체에 대한 프록시가 생성되었다 해도 그 객체의 메서드에 프록시 로직이 적용되려면 해당 메서드에대한 포인트컷이 참이어야 한다. 즉, 프록시를 생성할 때 포인트컷을 검사는 것과 메서드를 실행할 때 포인트컷을 검사하는 것은 별개이다.
* 어드바이저뿐만 아니라 @Aspect도 자동으로 인식해서 어드바이저를 만들고 프록시를 생성해준다.
## @Aspect
```java
@Slf4j
@Aspect
@Component
public class LogAspect {
    
    //포인트컷(AspectJExpressionPointcut)
    @Around("execution(* hello.proxy.app..*(..))")
    //어드바이스
    public Object advice(ProceedingJoinPoint joinPoint) throws Throwable {

        try {
            //message = (클래스이름).(메서드(..))
            String message = joinPoint.getSignature().toShortString();
            log.info("{} 호출", message);

            Object result = joinPoint.proceed();

            return result;
        } catch (Exception e) {
            log.info("예외 발생");
            throw e;
        }
    }
}
```
* 프록시 생성 과정
  1. @Aspect가 붙은 클래스를 스프링 빈으로 등록하면 AnnotationAwareAspectJAutoProxyCreator가 자동으로 Advisor를 만들어 BeanFactoryAspectJAdvisorsBuilder에 캐싱한다.
  2. @Aspect가 붙은 모든 클래스를 스캔해 어드바이저를 만든 후, AnnotationAwareAspectJAutoProxyCreator는 모든 스프링 빈에 대하여 스프링 빈으로 등록된 어드바이저나 BeanFactoryAspectJAdvisorsBuilder에 캐싱된 어드바이저의 포인트컷이 하나라도 참이 된다면 해당 빈에 대한 프록시를 생성한다.
* @Around에는 포인트 컷으로 적용할 AspectJ 표현식을 넣는다.
* @Around가 붙은 메서드는 어드바이스로 기능한다.
* 하나의 Aspect에 여러 어드바이스와 포인트것이 함께 존재할 수 있다.