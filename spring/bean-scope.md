# 빈 스코프와 생명주기 콜백
## 빈 생명주기
스프링 컨테이너 생성 -> 스프링 빈 생성 -> 의존관계 주입 -> 초기화 콜백 -> 빈 사용 -> 소멸전 콜백 -> 빈 소멸
## 빈 스코프
* 싱글톤: 기본 스코프, 스프링 컨테이너의 시작과 종료까지 유지되는 가장 넓은 범위의 스코프이다.
* 프로토타입: 스프링 컨테이너는 프로토타입 빈의 생성과 의존관계 주입까지만 관여하고 더는 관리하지 않는 매우 짧은 범위의 스코프이다.
* 웹 관련 스코프
    + request: 웹 요청이 들어오고 나갈때 까지 유지되는 스코프이다.
    + session: 웹 세션이 생성되고 종료될 때 까지 유지되는 스코프이다.
    + application: 웹의 서블릿 컨텍스트와 같은 범위로 유지되는 스코프이다.
## 스프링 빈의 생명주기 콜백
스프링은 크게 3가지 방법으로 빈 생명주기 콜백을 지원한다.
* 인터페이스(InitializingBean, DisposableBean)
* 설정 정보에 초기화 메서드, 소멸 메서드 지정
* @PostConstruct, @PreDestroy 어노테이션 지원
### 인터페이스 InitializingBean, DisposableBean
```java
public class InterfaceExample implements InitializingBean, DisposableBean {
    
    //서비스 시작 시 호출
    @Override
    public void afterPropertiesSet() throws Exception {
        System.out.println("bean created");
    }

    //서비스 종료시 호출
    @Override
    public void destroy() throws Exception {
        System.out.println("bean destroyed");
    }
}
```
* InitializingBean 은 afterPropertiesSet() 메서드로 초기화를 지원한다.
* DisposableBean 은 destroy() 메서드로 소멸을 지원한다.
* 이 인터페이스는 스프링 전용 인터페이스다. 해당 코드가 스프링 전용 인터페이스에 의존한다.
* 초기화, 소멸 메서드의 이름을 변경할 수 없다.
* 내가 코드를 고칠 수 없는 외부 라이브러리에 적용할 수 없다.
* __거의 사용하지 않는다.__
### 설정 정보에 초기화 메서드, 소멸 메서드 지정
```java
@Configuration
static class LifeCycleConfig {
    //초기화 메서드, 소멸 메서드 지정
    @Bean(initMethod = "beanCreated", destroyMethod = "beanDestroyed")
    public ConfigExample configExample() {
        return new ConfigExample();
    }
}
```
```java
public class ConfigExample {
    
    //서비스 시작 시 호출
    public void beanCreated(){
        System.out.println("bean created");
    }

    //서비스 종료시 호출
    public void beanDestroyed(){
        System.out.println("bean destroyed");
    }
}
```
* 메서드 이름을 자유롭게 줄 수 있다.
* 스프링 빈이 스프링 코드에 의존하지 않는다.
* 코드가 아니라 설정 정보를 사용하기 때문에 코드를 고칠 수 없는 외부 라이브러리에도 초기화, 소멸 메서드를 적용할 수 있다.
* 종료 메서드는 따로 적어주지 않아도 destroyMethod 는 기본값이 (inferred)으로 등록되어 있기 때문에 이 추론 기능은 close , shutdown 라는 이름의 메서드를 자동으로 호출해준다.
### @PostConstruct, @PreDestroy 어노테이션 지원
```java
public class AnnotationExample {
    //서비스 시작 시 호출
    @PostConstruct
    public void beanCreated(){
        System.out.println("bean created");
    }

    //서비스 종료시 호출
    @PreDestroy
    public void beanDestroyed(){
        System.out.println("bean destroyed");
    }
}
```
* __최신 스프링에서 가장 권장하는 방법이다.__
* 어노테이션 하나만 붙이면 되므로 매우 편리하다.
* 스프링에 종속적인 기술이 아니라 JSR-250
라는 자바 표준이다. 따라서 스프링이 아닌 다른 컨테이너에서도 동작한다.
* 단점은 외부 라이브러리에는 적용하지 못한다는 것이다. 외부 라이브러리를 초기화, 소멸 콜백을 해야 하면 @Bean의 기능을 사용해야한다.