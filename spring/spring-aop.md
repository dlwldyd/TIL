# 스프링 AOP
## @Aspect
```java
@Slf4j
@Aspect
public class AspectOrder {

    @Around("execution(* com.example.aop.order..*(..))")
    public Object doLog(ProceedingJoinPoint joinPoint) throws Throwable {
        //joinPoint.getSignature() : 반환타입, 패키지, 클래스이름, 파라미터 정보, 실행된 메서드 정보가 들어있음
        log.info("[log] {}", joinPoint.getSignature());
        return joinPoint.proceed();
    }
}
```
* @Aspect가 붙어있는 클래스가 횡단 관심사에 해당한다. 스프링 빈으로 등록하는 것이 필수이다.
* @Around 어노테이션의 값인 execution(* hello.aop.order..*(..)) 는 포인트컷이 된다. 이 값은 AspectJ 표현식인데 실제로 AspectJ를 사용하는게 아니라 AspectJ의 표현식만을 차용한 것이다.
* 포인트컷은 어느 클래스의 어느 메서드에 어드바이스를 적용할지의 기준이 된다.
* @Around가 붙어있는 메서드(doLog)는 어드바이스가 된다.
* 어드바이스는 실제 기능을 수행하는 로직에 덧붙일 부가기능로직이다.(횡단 관심사로써 적용할 로직)
## 포인트컷
```java
public class Pointcuts {

    @Pointcut("execution(* com.example.aop.order..*(..))")
    public void allOrder() {
    } //pointcut signature

    @Pointcut("execution(* *..*Service.*(..))")
    public void allService() {
    }

    @Pointcut("allOrder() && allService()")
    public void orderAndService() {
    }
}
```
```java
@Slf4j
@Aspect
public class AspectPointcut {

    // @Pointcut("execution(* com.example.aop.order..*(..))")
    // private void allOrder(){}

    // @Pointcut("execution(* *..*Service.*(..))")
    // private void allService(){}

    // @Around("allOrder()")
    @Around("com.example.aop.order.aop.Pointcuts.allOrder()")
    public Object doLog(ProceedingJoinPoint joinPoint) throws Throwable {
        log.info("[log] {}", joinPoint.getSignature());
        return joinPoint.proceed();
    }

    // @Around("allOrder() && allService()")
    @Around("com.example.aop.order.aop.Pointcuts.orderAndService()")
    public Object doTransaction(ProceedingJoinPoint joinPoint) throws Throwable {
        try {
            log.info("[트랜잭션 시작] {}", joinPoint.getSignature());
            Object result = joinPoint.proceed();
            log.info("[트랜잭션 커밋] {}", joinPoint.getSignature());
            return result;
        } catch (Exception e) {
            log.info("[트랜잭션 롤백] {}", joinPoint.getSignature());
            throw e;
        }finally {
            log.info("[리소스 릴리즈] {}", joinPoint.getSignature());
        }
    }
}
```
* 포인트컷은 어느 클래스의 어느 메서드에 어드바이스를 적용할지의 기준이 된다.
* @Around, @Before, @AfterReturning, @AfterThrowing, @After에 포인트컷을 지정할 수 있다.
* @Pointcut 어노테이션을 사용하면 포인트컷을 분리할 수 있다.
* 포인트컷을 분리해 따로 모듈화할 수 있다. 단, 포인트컷을 모듈화하면 포인트컷을 지정할 때 전체 패키지 경로를 적어주어야 한다.
* &&(AND), ||(OR), !(NOT)을 사용해 포인트컷을 조합할 수 있다.
## 어드바이스
### 어드바이스 적용 순서
```java
@Slf4j
public class AspectOrder {

    @Aspect
    @Order(2)
    public static class LogAspect {
        @Around("execution(* com.example.aop.order..*(..))")
        public Object doLog(ProceedingJoinPoint joinPoint) throws Throwable {
            log.info("[log] {}", joinPoint.getSignature());
            return joinPoint.proceed();
        }
    }

    @Aspect
    @Order(1)
    public static class TxAspect {
        @Around("execution(* com.example.aop.order..*(..))")
        public Object doTransaction(ProceedingJoinPoint joinPoint) throws Throwable {
            try {
                log.info("[트랜잭션 시작] {}", joinPoint.getSignature());
                Object result = joinPoint.proceed();
                log.info("[트랜잭션 커밋] {}", joinPoint.getSignature());
                return result;
            } catch (Exception e) {
                log.info("[트랜잭션 롤백] {}", joinPoint.getSignature());
                throw e;
            }finally {
                log.info("[리소스 릴리즈] {}", joinPoint.getSignature());
            }
        }
    }
}
```
* @Order 어노테이션을 사용해서 어드바이스의 적용 순서를 지정할 수 있다. 단, @Order는 클래스 레벨에서만 사용할 수 있다. 그렇기 때문에 하나의 Aspect에 여러 어드바이스가 있고 거기에 순서를 부여하고 싶다면 각각을 서로 다른 Aspect로 분리해야한다.
* `클라이언트` -> `doTransaction` -> `doLog` -> `비즈니스 로직` -> `doLog` -> `doTransction` -> `클라이언트`
### @Around
```java
@Slf4j
@Aspect
public class AspectV6Advice {

    @Around("execution(* com.example.aop.order..*(..))")
    public Object doTransaction(ProceedingJoinPoint joinPoint) throws Throwable {
        try {

            //@Before
            log.info("[트랜잭션 시작] {}", joinPoint.getSignature());
            Object result = joinPoint.proceed();

            //@AfterReturning
            log.info("[트랜잭션 커밋] {}", joinPoint.getSignature());
            return result;
        } catch (Exception e) {

            //@AfterThrowing
            log.info("[트랜잭션 롤백] {}", joinPoint.getSignature());
            throw e;
        }finally {

            //@After
            log.info("[리소스 릴리즈] {}", joinPoint.getSignature());
        }
    }
}
```
* 가장 강력한 어드바이스로 @Before, @AfterReturning, @AfterThrowing, @After의 모든 기능을 다 할 수 있고, 반환값을 바꿔치기할 수도 있고 예외를 바꿔치기 할 수도 있다.
* 파라미터로 ProceedingJoinPoint를 필수적으로 받아야 하며, ProceedingJoinPoint.proceed()를 호출하지 않는다면 타겟을 호출하지 않는다.(일부러 타겟을 호출하지 않는 경우가 아닌 한 무조건 넣어줘야 한다.)
### @Before
```java
@Before("execution(* com.example.aop.order..*(..))")
public void doBefore(JoinPoint joinPoint) { //ProceedingJoinPoint 사용 불가
    log.info("[before] {}", joinPoint.getSignature());
    //자동으로 타겟 실행
}
```
* 타겟이 호출되기 전에 호출도니다.
* ProceedingJoinPoint.proceed()를 마지막에 자동으로 호출한다.
### @AfterReturning
```java
@AfterReturning(value = "execution(* com.example.aop.order..*(..))", returning = "result")
public void doReturn(JoinPoint joinPoint, Object result) {
    log.info("[return] {} result={}", joinPoint.getSignature(), result);
}
```
* 타겟 호출 후 값이 리턴되었을 때 호출된다.
* 반환값을 바꿔치기 할 수는 없다.
* @AfterReturning의 returning값을 세팅하고 세팅된 값과 같은 이름의 파라미터를 받으면 그 값에 타겟 호출 후 반환된 값이 들어간다. 만약 파라미터의 타입을 반환값을 받을 수 없는 타입으로 지정하면 해당 어드바이스 자체가 호출되지 않는다.(doReturn을 실행 안함)
### @AfterThrowing
```java
@AfterThrowing(value = "execution(* com.example.aop.order..*(..))", throwing = "ex")
public void doThrowing(JoinPoint joinPoint, Exception ex) {
    log.info("[ex] {} message={}", ex, ex.getMessage());
    //Throw e 가 자동으로 됨
}
```
* 타겟 호출 후 예외가 throw 됬을 때 호출된다.
* 예외를 바꿔치기 할 수는 없다.
* @AfterThrowing의 throwing값을 세팅하고 세팅된 값과 같은 이름의 파라미터를 받으면 그 값에 타겟 호출 후 예외가 발생했을 때 throw된 예외가 들어간다. 만약 파라미터의 타입을 throw된 예외를 받을 수 없는 타입으로 지정하면 해당 어드바이스 자체가 호출되지 않는다.(doThrowing을 실행 안함)
* 마지막에 throw ex를 자동을 실행한다.
### @After
```java
@After("execution(* com.example.aop.order..*(..))")
public void doAfter(JoinPoint joinPoint) {
    log.info("[after] {}", joinPoint.getSignature());
}
```
* 메서드 실행이 종료되면 실행된다.(finally와 같다고 보면 된다.)
* 일반적으로 리소스를 해제할 때 사용한다.
### 전체 실행 순서
`@Around의 @Before에 해당하는 부분` -> `@Before` -> `비즈니스 로직` -> `@AfterReturning(예외 발생X), @AfterThrowing(예외 발생시)` -> `@After` -> `@Around의 @AfterReturning에 해당하는 부분(예외 발생 X), @Around의 @AfterThrowing에 해당하는 부분(예외 발생시)` -> `@Around의 @After에 해당하는 부분`
## 내부 호출 문제
```java
@Slf4j
@Component
public class CallService {

    public void external() {
        log.info("call external");
        internal(); //내부 메서드 호출(this.internal())
    }

    public void internal() {
        log.info("call internal");
    }
}
```
```java
@Slf4j
@Aspect
public class CallLogAspect {

    @Before("execution(* hello.aop.internalcall..*.*(..))")
    public void doLog(JoinPoint joinPoint) {
        log.info("aop={}", joinPoint.getSignature());
    }
}
```
* CallService의 external() 메서드와 internal() 메서드는 모두 조인포인트이다. 하지만 external()은 internal()을 호출하고 이는 this.internal()과 같다. 즉, 프록시의 internal()이 아닌 타겟의 internal()을 호출한다. 그렇기 때문에 external()에서 호출하는 internal()에는 어드바이스가 적용되지 않는다.
### ObjectProvider를 통한 해결
```java
@Slf4j
@Component
public class CallService {

    private final ObjectProvider<CallService> callServiceProvider;

    public CallService(ObjectProvider<CallService> callServiceProvider) {
        this.callServiceProvider = callServiceProvider;
    }

    public void external() {
        log.info("call external");
        CallService callService = callServiceProvider.getObject();
        callService.internal();
    }

    public void internal() {
        log.info("call internal");
    }
}
```
* ObjectProvider를 통해 CallService 클래스로 등록된 스프링 빈(프록시)을 추출한다. 그리고 내부 호출을 할 때 추출한 프록시의 메서드(internal)를 호출하여 어드바이스가 적용되도록 한다.
### 구조 변경을 통한 해결
```java
@Slf4j
@Component
public class CallService {

    private final InternalService internalService;

    public CallService(InternalService internalService) {
        this.internalService = internalService;
    }

    public void external() {
        log.info("call external");
        internalService.internal();
    }
}
```
```java
@Slf4j
@Component
public class InternalService {
    public void internal() {
        log.info("call internal");
    }
}
```
* 클래스의 구조를 변경하여 내부호출을 하는 메서드를 다른 클래스로 분리할 수 있다.
```java
@Slf4j
@Component
public class CallServiceV0 {

    public void external() {
        log.info("call external");
    }

    public void internal() {
        log.info("call internal");
    }
}
```
```java
@SpringBootTest
class CallServiceV4Test {

    @Autowired
    CallServiceV4 callServiceV4;

    @Test
    void external() {
        call();
    }

    private void call() {
        callServiceV4.external();
        callServiceV4.internal();
    }
}
```
* 메서드가 내부 호출을 하지않도록 하나의 메서드를 여러개의 메서드로 분리하여 클라이언트가 각각의 메서드를 호출하도록 구조를 변경할 수도 있다.