# AspectJ 포인트컷 표현식
## execution
```
execution(접근제어자? 반환타입 선언타입?메서드이름(파라미터) 예외?)
```
* ?가 붙은 부분은 생략 가능하다.
* 선언타입을 쓸 때는 패키지까지 써야한다.
* '*'이나 '..'같은 패턴을 지정할 수 있다.
* 선언타입을 매칭할 때 선언타입의 자식타입은 포인트컷 대상이 된다. 단, 자식 타입의 메서드 중 선언타입에 있지 않은 메서드는 포인트컷 대상이 되지 않는다.
* 파라미터를 매칭할 때 파라미터 타입의 자식타입은 포인트컷 대상이 되지 않는다.(정확히 같은 타입이어야 한다. 인터페이스 사용 불가)
* 가장 중요하고 자주 쓰인다.
### 예시
```
execution(public String hello.aop.member.MemberServiceImpl.hello(String))
```
* public으로 선언되고 String 타입을 반환하며 hello.aop.member 패키지에 있는 MemberServiceImpl 클래스의 String 타입의 파라미터가 하나 들어오는 hello 메서드
```
execution(* *(..))
```
* 모든 클래스의 모든 메서드
```
execution(* hello(..))
```
* 모든 클래스의 이름이 hello인 모든 메서드
```
execution(* hel*(..))
```
* 모든 클래스의 이름이 hel로 시작하는 모든 메서드
```
execution(* hello.aop.member.*.*(..))
```
* hello.aop.member 패키지에 있는 모든 클래스의 모든 메서드
```
execution(* hello.aop.member..*.*(..))
```
* hello.aop.member 패키지와 그 하위 패키지에 있는 모든 클래스의 모든 메서드
```
execution(* *(String))
```
* 모든 클래스의 String 타입의 파라미터 하나만 있는 모든 메서드
```
execution(* *(*))
```
* 모든 클래스의 파라미터가 하나만 있는 모든 메서드
```
execution(* *(String, ..))
```
* 모든 클래스의 첫 번째 파라미터가 String 타입인 모든 메서드
## within
* 특정 타입의 조인포인트를 매칭한다.
* execution의 선언타입 부분만 사용한다고 보면 된다.
* 자주 사용하지는 않는다.
* 부모타입이 매칭되어도 포인트컷 대상이 되지 않는다.(정확히 같은 타입이어야 한다. 인터페이스 사용 불가)
### 예시
```
within(hello.aop.member.MemberServiceImpl)
```
* MemberServiceImpl의 모든 메서드
```
within(hello.aop.member.*Service*)
```
* hello.aop.member 패키지에 있는 Service라는 단어가 들어가 있는 클래스의 모든 메서드
```
within(hello.aop..*)
```
* hello.aop 패키지와 그 하위 패키지에 있는 모든 클래스의 모든 메서드
## args
* execution의 파라미터 부분과 같다고 보면 된다.
* execution과의 차이점은 execution은 파라미터의 타입의 자식 탕입은 포인트컷 대상이 되지 않지만(정확히 같은 타입이어야 함) args에서는 파라미터 타입의 자식 타입이 포인트컷의 대상이 된다.
* args혼자 단독으로 사용해서는 안된다.(에러남)
### 예시
```
args(String)
```
* String 타입의 파라미터 하나만 있는 모든 메서드
```
args(Object)
```
* Object 타입의 파라미터 하나만 있는 모든 메서드
```
args(String, *)
```
* 첫 번째 파라미터가 String 타입이고 파라미터의 개수가 2개인 모든 메서드
```
args(String, ..)
```
* 첫 번째 파라미터가 String 타입인 모든 메서드
## @target, @within
* @target과 @within은 클래스를 타겟으로 하는 어노테이션이 있는지를 보고 조인포인트로 적용할지 정한다.
* @target은 어노테이션이 붙은 클래스의 부모클래스의 메서드까지 조인포인트가 되지만 @within은 어노테이션이 붙은 클래스의 메서드만 조인포인트가 된다.
* @target 혼자 사용해서는 안된다.(에러남)
* 잘 사용하지는 않는다.
### 예시
```java
@Target(ElementType.TYPE)
@Retention(RetentionPolicy.RUNTIME)
public @interface ClassAop {
}
```
```java
public class Parent {
    public void parentMethod(){} //부모에만 있는 메서드
}
```
```java
@ClassAop
public class Child extends Parent {
    public void childMethod(){}
}
```
```java
@Slf4j
@Aspect
public class AtTargetAtWithinAspect {

    //@target: 인스턴스 기준으로 모든 메서드의 조인 포인트를 선정, 부모 타입의 메서드도 적용
    @Around("execution(* hello.aop..*(..)) && @target(hello.aop.member.annotation.ClassAop)")
    public Object atTarget(ProceedingJoinPoint joinPoint) throws Throwable
    {
        log.info("[@target] {}", joinPoint.getSignature());
        return joinPoint.proceed();
    }

    //@within: 선택된 클래스 내부에 있는 메서드만 조인 포인트로 선정, 부모 타입의 메서드는 적용되지 않음
    @Around("execution(* hello.aop..*(..)) && @within(hello.aop.member.annotation.ClassAop)")
    public Object atWithin(ProceedingJoinPoint joinPoint) throws Throwable
    {
        log.info("[@within] {}", joinPoint.getSignature());
        return joinPoint.proceed();
    }
}
```
* @ClassAop는 Child타입에 붙어있다. 이 때 @target(hello.aop.member.annotation.ClassAop)는 parentMethod()와 childMethod()모두 조인포인트로 선정하지만, @within(hello.aop.member.annotation.ClassAop)는 childMethod()만 조인포인트로 선정한다.
## @annotation
* 메서드를 타겟으로 하는 어노테이션이 있는지를 보고 조인포인트로 선정할지 결정한다.
### 예시
```java
@Component
public class MemberServiceImpl implements MemberService {

    @MethodAop("test value")
    @Override
    public String hello(String param) {
        return "ok";
    }

    public String internal(String param) {
        return "ok";
    }
}
```
```java
@Slf4j
@Aspect
public class AtAnnotationAspect {

    @Around("@annotation(hello.aop.member.annotation.MethodAop)")
    public Object doAtAnnotation(ProceedingJoinPoint joinPoint) throws Throwable {
        log.info("[@annotation] {}", joinPoint.getSignature());
        return joinPoint.proceed();
    }
}
```
* MethodAop 어노테이션이 붙은 hello()만 조인포인트로 선정되고 internal은 조인포인트로 선정되지 않는다.
## bean
* 스프링 전용 포인트컷 지시자이다. AspectJ에서는 사용되지 않는다.
* 스프링 빈의 이름으로 조인포인트를 선정한다.
### 예시
```java
@Slf4j
@Aspect
public class BeanAspect {
    @Around("bean(orderService) || bean(*Repository)")
    public Object doLog(ProceedingJoinPoint joinPoint) throws Throwable {
        log.info("[bean] {}", joinPoint.getSignature());
        return joinPoint.proceed();
    }
}
```
* 스프링 빈의 이름이 orderService인 빈과 Repository라는 이름으로 끝나는 빈의 모든 메서드가 조인포인트로 선정된다.
## this, target
* this와 target에는 선언 타입을 넣는다.
* spring.aop.proxy-target-class=false은 인터페이스가 있을 때는 JDK 동적 프록시로, 없을 때는 CGLIB로 프록시를 생성하라는 옵션이다.(default로는 항상 CGLIB로 프록시를 생성한다.)
```properties
spring.aop.proxy-target-class=false
```
```java
@Slf4j
@Aspect
public class ThisTargetAspect {

    @Around("this(hello.aop.member.MemberService)")
    public Object doThisInterface(ProceedingJoinPoint joinPoint) throws Throwable {
        log.info("[this-interface] {}", joinPoint.getSignature());
        return joinPoint.proceed();
    }

    @Around("target(hello.aop.member.MemberService)")
    public Object doTargetInterface(ProceedingJoinPoint joinPoint) throws Throwable {
        log.info("[target-interface] {}", joinPoint.getSignature());
        return joinPoint.proceed();
    }

    @Around("this(hello.aop.member.MemberServiceImpl)")
    public Object doThis(ProceedingJoinPoint joinPoint) throws Throwable {
        log.info("[this-impl] {}", joinPoint.getSignature());
        return joinPoint.proceed();
    }

    @Around("target(hello.aop.member.MemberServiceImpl)")
    public Object doTarget(ProceedingJoinPoint joinPoint) throws Throwable {
        log.info("[target-impl] {}", joinPoint.getSignature());
        return joinPoint.proceed();
    }
}
```
* this는 프록시를 대상으로 포인트컷 판별한다.(프록시 대상이라기 보다는 현재 스프링빈으로 등록된 객체(this)를 대상으로 한다.) 프록시 생성시에는 프록시가 없기 때문에 target 을 대상으로 포인트컷 판별 후 프록시를 생성하지만 프록시 생성 후에는 스프링빈으로 프록시가 등록되어 있기 때문에 프록시를 대상으로 포인트컷을 판별한다.
* target은 실제 기능을 하는 객체(target)을 대상으로 포인트컷을 판별한다.
## 매개변수 전달
```java
@Around("execution(* hello.aop.member..*.*(..))")
public Object logArgs1(ProceedingJoinPoint joinPoint) throws Throwable {
    Object arg1 = joinPoint.getArgs()[0];
    log.info("[logArg1]{} arg1={}", joinPoint.getSignature(), arg1);
    return joinPoint.proceed();
}
```
* joinPoint.getArgs()함수로 배열의 형태로 조인포인트의 파라미터에 접근할 수 있다.
```java
@Around("execution(* hello.aop.member..*.*(..)) && args(arg, ..)")
public Object logArgs2(ProceedingJoinPoint joinPoint, Object arg) throws Throwable {
    log.info("[logArg2]{} arg={}", joinPoint.getSignature(), arg);
    return joinPoint.proceed();
}
```
* args에 파라미터 타입 대신 파라미터 이름을 넣고 어드바이스에 같은 이름의 파라미터를 받으면 조인포인트의 파라미터에 접근할 수 있다. 단, 어드바이스의 파라미터 타입이 조인포인트의 파라미터를 받을 수 없는 타입이면 어드바이스는 실행되지 않는다.(조인포인트로 선정X)
* args(arg, ..) -> args(String, ..)와 같다.
```java
@Before("execution(* hello.aop.member..*.*(..)) && this(obj)")
public void thisArgs(JoinPoint joinPoint, MemberService obj) {
    log.info("[this]{}, obj={}", joinPoint.getSignature(), obj.getClass());
}
```
* obj에 프록시가 들어온다.(스프링 빈으로 등록된 객체가 들어온다.)
* 어드바이스의 파라미터(obj) 타입으로 등록된 스프링 빈(프록시)를 가져온다.
* this(obj) -> this(hello.aop.member.MemberService)와 같다.
```java
@Before("execution(* hello.aop.member..*.*(..)) && target(obj)")
public void targetArgs(JoinPoint joinPoint, MemberService obj) {
    log.info("[target]{}, obj={}", joinPoint.getSignature(), obj.getClass());
}
```
* obj에 실제로 호출하는 대상(target)이 들어온다.
* 어드바이스의 파라미터(obj) 타입으로 등록된 스프링 빈이 실제로 호출하는 대상(target)을 가져온다.
* target(obj) -> target(hello.aop.member.MemberService)와 같다.
```java
@Before("execution(* hello.aop.member..*.*(..)) && @target(annotation)")
public void atTarget(JoinPoint joinPoint, ClassAop annotation) {
    log.info("[@target]{}, annotation={}", joinPoint.getSignature(), annotation);
}
```
```java
@Before("execution(* hello.aop.member..*.*(..)) && @within(annotation)")
public void atWithin(JoinPoint joinPoint, ClassAop annotation) {
    log.info("[@within]{}, annotation={}", joinPoint.getSignature(), annotation);
}
```
* 조인포인트의 클래스에 붙어있는 어노테이션을 가져온다.
* 어드바이스의 파라미터(annotation) 타입의 어노테이션을 받는다. 그리고 해당 어노테이션(ClassAop)이 붙어있는 클래스의 모든 메서드가 조인포인트로 선정된다.
* 어노테이션에 들어있는 값을 가져올 수도 있다.
* @target(annotation) -> @target(hello.aop.member.annotation.ClassAop)와 같다.
* @within(annotation) -> @within(hello.aop.member.annotation.ClassAop)와 같다.
```java
@Target(ElementType.METHOD)
@Retention(RetentionPolicy.RUNTIME)
public @interface MethodAop {
    String value();
}
```
```java
@Before("execution(* hello.aop.member..*.*(..)) && @annotation(annotation)")
public void atAnnotation(JoinPoint joinPoint, MethodAop annotation) {
    log.info("[@annotation]{}, annotationValue={}", joinPoint.getSignature(), annotation.value());
}
```
* 조인포인트에 붙어있는 어노테이션을 가져온다.
* 어드바이스의 파라미터(annotation) 타입의 어노테이션을 받는다.(안에 들어있는 값까지) 그리고 해당 어노테이션(MethodAop)이 붙어있는 메서드만 조인포인트로 선정된다.
* 어노테이션에 들어있는 값을 가져올 수도 있다.
* @annotation(annotation) -> @annotation(hello.aop.member.annotation.MethodAop)와 같다.