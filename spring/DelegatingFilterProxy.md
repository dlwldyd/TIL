# DelegatingFilterProxy, FilterChainProxy
<center><img src="./../img/SecurityFilterChain.png"></center>

* 서블릿 필터에서는 스프링 빈을 주입받지 못하고 스프링 기술을 사용할 수도 없다 왜냐하면 서블릿 필터는 서블릿 컨테이너에서 생성되고 스프링 빈은 스프링 컨테이너에서 생성되기 때문에 전혀 다른 영역에 있기 때문이다. 그래서 스프링 시큐리티에서는 스프링 빈을 주입받고 스프링 기술을 사용하기 위해 프록시 기술을 사용하여 보안 관련 처리를 위임하는 방법을 사용하였다.
* 실제 서블릿 필터에는 DelegatingFilterProxy가 등록되어 있다.(springSecurityFilterChain라는 이름으로 등록됨) 그리고 DeligatingFilterProxy는 스프링 빈으로 등록되어 있는 FilterChainProxy 객체에게 보안 관련 처리를 위임한다. 이때 FilterChainProxy 객체는 DelegaingFilterProxy처럼 springSecurityFilterChain이라는 이름으로 스프링 빈에 등록되어있다.
* 동작 순서
  1. 클라이언트로 부터 요청이 오면 서블릿 필터가 순차적으로 실행되며 서블릿 필터로 등록되어있는 DelegatingFilterProxy를 실행하게 된다.
  2. DelegatingFilterProxy는 springSecurityFilterChain라는 이름으로 등록된 스프링 빈(FilterChainProxy 객체)을 찾아 doFilter()를 실행한다.
  3. FilterChainProxy에 등록된 스프링 시큐리티 필터를 순차적으로 실행한다.
  4. 모든 스프링 시큐리티 필터 처리를 완료했으면(보안에 통과할 경우) DelegatingFilterProxy 다음의 필터를 순차적으로 실행한 후 DispatcherServlet을 실행해 클라이언트의 요청을 처리한다.