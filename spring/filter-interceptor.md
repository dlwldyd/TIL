# 서블릿 필터와 인터셉터
## 서블릿 필터
<center><img src="./../img/servlet-filter.png"></center>

* 필터를 적용하면 필터가 호출 된 다음에 서블릿이 호출된다. 스프링의 경우에는 여기에서의 서블릿을 Dispatcher Servlet으로 생각하면 된다.
* 필터는 체인으로 구성되는데, 중간에 필터를 자유롭게 추가할 수 있다.
```java
@Slf4j
public class LogFilter implements Filter {

    @Override
    public void init(FilterConfig filterConfig) throws ServletException {
        Filter.super.init(filterConfig);
        log.info("log filter init");
    }

    @Override
    public void doFilter(ServletRequest request, ServletResponse response, FilterChain chain) throws IOException, ServletException {
        log.info("log filter doFilter");

        HttpServletRequest httpRequest = (HttpServletRequest) request;
        String requestURI = httpRequest.getRequestURI();

        String uuid = UUID.randomUUID().toString();

        try {
            log.info("REQUEST [{}] [{}]", uuid, requestURI);
            //다음 필터 호출, 다음 필터가 없고 필터링 되지 않으면 DispatcherServlet 호출
            chain.doFilter(request, response);
        } catch (Exception e) {
            throw e;
        }finally {
            //Controller 에 의해 response 가 반환되고 DispatcherServlet 의 호출이 끝나면 실행
            log.info("RESPONSE [{}] [{}]", uuid, requestURI);
        }
    }

    @Override
    public void destroy() {
        Filter.super.destroy();
        log.info("log filter destroy");
    }
}
```
* Filter 인터페이스를 상속받아야 한다.
* init(): 필터 초기화 메서드, 서블릿 컨테이너가 생성될 때 호출된다.
* doFilter(): 클라이언트의 요청이 올 때 마다 해당 메서드가 호출된다. 필터의 로직을 구현하면 된다.
* destroy(): 필터 종료 메서드, 서블릿 컨테이너가 종료될 때 호출된다.
* chain.doFilter(request, response) : 다음 필터가 있으면 필터를 호출하고, 필터가 없으면 서블릿을 호출한다. 만약 이 로직을 호출하지 않으면 다른 필터와 컨트롤러를 호출하지 않고 response를 내보낸다. 다음 필터 또는 서블릿을 호출할 때 request , response를 다른 객체로 바꿀 수 있다.
```java
@Configuration
public class WebConfig {
    
    @Bean
    public FilterRegistrationBean logFilter() {
        FilterRegistrationBean<Filter> filterRegistrationBean = new FilterRegistrationBean<>();

        filterRegistrationBean.setFilter(new LogFilter());
        
        filterRegistrationBean.setOrder(1);

        filterRegistrationBean.addUrlPatterns("/*");

        return filterRegistrationBean;
    }
}
```
* filterRegistrationBean.setFilter(new LogFilter()) : 내가 만든 서블릿 필터를 넣어야 한다.
* filterRegistrationBean.setOrder(1) : 필터의 적용순서를 정해야 한다.
* filterRegistrationBean.addUrlPatterns("/*") : 적용할 URL 을 정해야 한다. 여러 URL 을 넣을 수 있다.
* @WebFilter 어노테이션을 통해 컴포넌트 스캔으로 등록할 수도 있다. 다만 여러개의 필터를 사용하는 경우 유지보수 하기에는 별로 좋은 방법이 아니다.
## 스프링 인터셉터
<center><img src="./../img/spring-interceptor.png"></center>

* 스프링 인터셉터는 Dispatcher Servlet과 컨트롤러 사이에서 컨트롤러 호출 직전에 호출 된다.
* 스프링 인터셉터는 서블릿 필터보다 URL패턴을 더 정밀하게 설정할 수 있다.
```java
@Slf4j
public class LogInterceptor implements HandlerInterceptor {


    public static final String LOG_ID = "logId";

    @Override
    public boolean preHandle(HttpServletRequest request, HttpServletResponse response, Object handler) throws Exception {

        String requestURI = request.getRequestURI();
        String uuid = UUID.randomUUID().toString();

        request.setAttribute(LOG_ID, uuid);

        //@RequestMapping 사용 시 HandlerMethod 가 넘어옴
        //정적 리소스 사용 시 ResourceHttpRequestHandler 가 넘어옴
        if (handler instanceof HandlerMethod) {
            
            HandlerMethod hm = (HandlerMethod) handler;
        }

        log.info("REQUEST [{}] [{}] [{}]", uuid, requestURI, handler);

        return true;
    }

    @Override
    public void postHandle(HttpServletRequest request, HttpServletResponse response, Object handler, ModelAndView modelAndView) throws Exception {
        log.info("postHandle [{}]", modelAndView);
    }

    @Override
    public void afterCompletion(HttpServletRequest request, HttpServletResponse response, Object handler, Exception ex) throws Exception {
        String requestURI = request.getRequestURI();
        String logId = (String) request.getAttribute(LOG_ID);

        log.info("RESPONSE [{}] [{}] [{}]", logId, requestURI, handler);
        if (ex != null) {
            //오류는 {}를 안넣어도 됨
            log.error("afterCompletion error!!", ex);
        }
    }
}
```
* preHandle : 컨트롤러 호출 전에 호출된다. preHandle 의 응답값이 true 이면 다음으로 진행하고(다음 인터셉터나 컨트롤러), false 이면 더는 진행하지 않는다.
* postHandle : 컨트롤러 호출 후에 호출된다.  컨트롤러에서 예외가 발생하면 postHandle 은 호출되지 않는다.
* afterCompletion : 뷰가 렌더링 된 이후에 호출된다. 컨트롤러에서 예외가 발생해도 항상 호출된다. 이 경우 예외(ex)를 파라미터로 받아서 어떤 예외가 발생했는지 로그로 출력할 수 있다.
```java
@Configuration
public class WebConfig implements WebMvcConfigurer {
    
    @Override
    public void addInterceptors(InterceptorRegistry registry) {
        registry.addInterceptor(new LogInterceptor())
                .order(1)
                .addPathPatterns("/**")
                .excludePathPatterns("/css/**", "/*.ico", "/error");
    }
}
```
* 스프링 인터셉터를 등록하려면 WebMvcConfigurer 인터페이스를 상속받고 addInterceptor를 오버라이드 해야한다.
* registry.addInterceptor(new LogInterceptor()) : 내가 만든 스프링 인터셉터를 추가한다.
* .order(1) : 스프링 인터셉터의 적용순서를 정한다.
* .addPathPatterns("/**") : 적용할 URL 을 정해야 한다. 여러 URL 을 넣을 수 있다.
* .excludePathPatterns("/css/**", "/*.ico", "/error") : 적용하지 않을 URL 을 정할 수 있다. 여러 URL 을 넣을 수 있다.