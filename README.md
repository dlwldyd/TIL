# Today I Learned(TIL)
## Spring
* [IoC(Inversion of Control)](spring/IoC(Inversion%20of%20Control).md)
* [빈 스코프와 생명주기 콜백](spring/bean-scope.md)
* [Servlet](spring/servlet.md)
* [검증](spring/validation.md)
* [쿠키, 세션 처리](spring/spring-cookie-session.md)
* [HTTP 요청 매핑](spring/http-request-mapping.md)
* [타입 컨버터](spring/type-converter.md)
* [파일 업로드](spring/file-upload.md)
* [Mockito](spring/mockito.md)
* [FactoryBean](spring/FactoryBean.md)
* [JDBC](spring/JDBC.md)
* [프로파일 나누기](spring/profiles.md)
* [WebClient](spring/WebClient.md)
* [Vault](spring/vault.md)
* [logback](spring/logback.md)
* [http body 값 여러번 읽기(ContentCachingRequestWrapper)](spring/ContentCachingRequestWrapper)
* [BigQuery](spring/bigquery.md)
* __DI(Dependency Injection)__
    + [DI를 사용하는 이유](spring/DI(Dependency%20Injection).md)
    + [컴포넌트 스캔에서 의존관계 주입 방법](spring/component_scan.md)
* __스프링 MVC__
  + [스프링 MVC 동작 흐름](spring/spring-mvc-flow.md)
  + [메시지 컨버터, Argument Resolver](spring/message-converter.md)
  + [서블릿 필터, 스프링 인터셉터](spring/filter-interceptor.md)
  + [오류, 예외처리](spring/spring-exception.md)
* __타임리프__
  + [타임리프 기본 사용법](spring/thymeleaf-uses.md)
  + [타임리프, 스프링 통합](spring/thymeleaf-spring-integration.md)
  + [메시지, 국제화](spring/messages.md)
* __AOP__
  + [빈 후처리기](spring/bean-postprocessor.md)
  + [동적 프록시](spring/dynamic-proxy.md)
  + [스프링 AOP](spring/spring-aop.md)
  + [AspectJ 포인트컷 표현식](spring/aspecj.md)
* __Spring Security__
  + [DelegatingFilterProxy, FilterChainProxy](spring/DelegatingFilterProxy.md)
  + [스프링 시큐리티 설정 API, 스프링 시큐리티 필터](spring/spring-security-filter.md)
  + [다중 보안 설정](spring/multi-config.md)
  + [SecurityContext](spring/SecurityContext.md)
  + [Form Login 인증 처리](spring/form-login.md)
  + [JSON 포맷 인증 처리](spring/ajax-login.md)
  + [JWT 기반 인증 처리](spring/jwt-login.md)
  + [커스텀 DSL](spring/custom-dsl.md)
  + [커스텀 SecurityMetadataSource, FilterSecurityInterceptor 등록하기](spring/FilterSecurityInterceptor.md)
  + [계층 권한 적용하기](spring/role-hierarchy.md)
  + [커스텀 AccessDecisionVoter 등록하기](spring/AccessDecisionVoter.md)
  + [메서드 보안 적용](spring/method-security.md)
  + [OAuth](spring/oauth.md)
  + [CORS](spring/cors.md)
  + [WebSecurityConfigurerAdapter deprecated](spring/WebSecurityConfigurerAdapter-deprecated.md)
* __Spring WebSocket__
  + [웹소켓, SockJS](spring/websocket.md)
  + [메세지 브로커](spring/message-broker.md)
  + [STOMP](spring/stomp.md)
* __Spring Cloud__
  + [넷플릭스 유레카](spring/spring-netflix-eureka.md)
  + [API Gateway](spring/api-gateway.md)
  + [Configuration Server](spring/configuration-server.md)
  + [Spring Cloud Bus](spring/spring-cloud-bus.md)
  + [OpenFeign](spring/OpenFeign.md)
## JPA
* [JPA 기본](jpa/jpa-base.md)
* [영속성 컨텍스트](jpa/persistence-context.md)
* [엔티티 매핑](jpa/entity-mapping.md)
* [연관관계 매핑](jpa/relationship-mapping.md)
* [상속관계 매핑, @MappedSuperclass](jpa/inheritance-mapping.md)
* [즉시 로딩, 지연 로딩](jpa/lazy-loading.md)
* [영속성 전이, 고아 객체](jpa/cascade.md)
* [값 타입](jpa/value-type.md)
* [JPQL](jpa/jpql.md)
* [이벤트 어노테이션](jpa/event-annotation.md)
* [OSIV](jpa/osiv.md)
* __Spring Data JPA__
  + [공통 인터페이스](jpa/common_jpa_interface.md)
  + [쿼리 메서드](jpa/query-mothod.md)
  + [사용자 정의 리포지토리](jpa/custom-repository.md)
  + [Auditing](jpa/auditing.md)
  + [페이징](jpa/paging.md)
  + [Persistable](jpa/persistable.md)
* __QueryDSL__
  + [QueryDSL 기본](jpa/querydsl-basic.md)
  + [프로젝션과 반환타입](jpa/projection.md)
  + [동적 쿼리](jpa/dynamic-query.md)
  + [벌크성 쿼리, SQL function](jpa/bulk-query.md)
  + [QueryDSL 사용해서 Page<> 반환](jpa/Querydsl-Page.md)
## 디자인 패턴
* [싱글톤 패턴](design-pattern/singleton-pattern.md)
* [프로토타입 패턴](design-pattern/prototype-pattern.md)
* [PRG 패턴](design-pattern/prg-pattern.md)
* [템플릿 메서드 패턴](design-pattern/template-method.md)
* [전략 패턴(템플릿 콜백 패턴)](design-pattern/strategy.md)
* [프록시 패턴](design-pattern/proxy-pattern.md)
* [데코레이터 패턴](design-pattern/decorator-pattern.md)
* [옵저버 패턴](design-pattern/observer-pattern.md)
* [빌더 패턴](design-pattern/builder-pattern.md)
* [팩토리 메서드 패턴](design-pattern/factory-method.md)
* [추상 팩토리 패턴](design-pattern/abstract-factory.md)
* [어댑터 패턴](design-pattern/adapter-pattern.md)
* [브릿지 패턴](design-pattern/bridge-pattern.md)
* [퍼사드 패턴](design-pattern/facade-pattern.md)
* [컴포지트 패턴](design-pattern/composite-pattern.md)
* [플라이웨이트 패턴](design-pattern/flyweight-pattern.md)
* [책임 연쇄 패턴](design-pattern/chain-of-responsibility.md)
* [커맨드 패턴](design-pattern/command-pattern.md)
* [인터프리터 패턴](design-pattern/interpreter-pattern.md)
* [이터레이터 패턴](design-pattern/iterator-pattern.md)
* [중재자 패턴](design-pattern/mediator-pattern.md)
* [메멘토 패턴](design-pattern/memento-pattern.md)
* [상태 패턴](design-pattern/state-pattern.md)
* [비지터 패턴](design-pattern/visitor-pattern.md)
## 웹
* [HTTP 기초](web/http.md)
* [HTTP 메서드](web/httpMethod.md)
* [HTTP 상태코드](web/httpStatusCode.md)
* [HTTP 헤더](web/httpHeader.md)
* [HTTP 쿠키와 세션](web/cookie_session.md)
* [HTTP 캐시](web/cache.md)
## JAVA
* [volatile](java/volatile.md)
* [Atomic 클래스](java/atomic.md)
* [ThreadLocal](java/threadlocal.md)
* [java.util.function](java/function.md)
* [인텔리제이 단축키](java/intelliJ.md)
* [직렬화(Serializable)](java/Serializable.md)
* [DocumentContext](java/DocumentContext.md)
* [JsonDeserializer, JsonPojoBuilder](java/jsonDeserializer.md)
## Go
* [Go 기본](Go/basic.md)
* [자료구조](Go/data-structure.md)
* [패키지](Go/package.md)
* [Echo](Go/echo.md)
## 프론트
* [JSX](front/JSX.md)
* [props](front/props.md)
* [React Hook](front/react-hook.md)
* [React Router](front/router.md)
* [styled components](front/styled-component.md)
* [webRTC](front/webRTC.md)
* [Recoil](front/recoil.md)
## 코딩테스트
* [코딩테스트](coding-test/coding-test.md)
## 기타
* [MongoDB](etc/mongoDB.md)
* [kubernetes](etc/kubernetes.md)