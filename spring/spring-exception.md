# 예외, 오류 처리
## 예외, 오류 처리 흐름
<center><img src="./../img/spring-exception.png"></center>

1. 클라이언트가 HTTP 요청을 보낸다.
2. 서버에서 오류가 발생해서 WAS까지 전파된다.
3. 지정해둔 오류페이지가 없으면 WAS에서 기본으로 사용하는 오류페이지를 응답하고 지정해둔 오류페이지가 있으면 해당 오류페이지를 요청한다.
4. 오류페이지를 제공하는 컨트롤러가 호출되고 오류페이지를 응답한다.
```java
public enum DispatcherType {
    FORWARD,
    INCLUDE,
    REQUEST,
    ASYNC,
    ERROR
}
```
```java
@Bean
public FilterRegistrationBean logFilter() {
    FilterRegistrationBean<Filter> filterRegistrationBean = new FilterRegistrationBean<>();
    filterRegistrationBean.setFilter(new LogFilter());
    filterRegistrationBean.setOrder(1);
    filterRegistrationBean.addUrlPatterns("/*");
    //filterRegistrationBean.setDispatcherTypes(DispatcherType.REQUEST, DispatcherType.ERROR);
    return filterRegistrationBean;
}
```
* 오류 발생 시 필터를 2번 호출하는 것은 비효율적일 수 있다. 그래서 서블릿 필터는 해당 요청이 클라이언트로부터 발생한 정상 요청인지 오류페이지를 요청하는 내부 요청인지 구분하기 위해 DispatcherType이라는 추가 정보를 사용한다.
* DispatcherType은 클라이언트로부터의 정상 요청일 때 REQUEST가 세팅되고 오류페이지에 대한 요청일 때 ERROR가 세팅된다.
* filterRegistrationBean.setDispatcherTypes()함수로 어떤 요청일 때 필터를 거칠지 정할 수 있는데 default는 DispatcherType.REQUEST이다.
```java
 @Override
public void addInterceptors(InterceptorRegistry registry) {
    registry.addInterceptor(new LogInterceptor())
            .order(1)
            .addPathPatterns("/**")
            .excludePathPatterns("/css/**", "/*.ico", "/error");
}
```
* 인터셉터의 경우에는 DispatcherType을 사용하지 않는다.
* 인터셉터는 서블릿 필터보다 더 정교한 URL패턴을 적용할 수 있는데 이를 통해 오류페이지를 요청하는 URL을 제외시키면 된다. 스프링부트는 기본적으로 `/error`라는 경로로 기본 오류페이지를 설정하기 때문에 `/error`를 제외시키면 된다.
## 오류 페이지
```java
@Controller
@RequestMapping("${server.error.path:${error.path:/error}}")
public class BasicErrorController extends AbstractErrorController {

    ...

    @RequestMapping(produces = MediaType.TEXT_HTML_VALUE)
	public ModelAndView errorHtml(HttpServletRequest request, HttpServletResponse response) {
		HttpStatus status = getStatus(request);
		Map<String, Object> model = Collections
				.unmodifiableMap(getErrorAttributes(request, getErrorAttributeOptions(request, MediaType.TEXT_HTML)));
		response.setStatus(status.value());
		ModelAndView modelAndView = resolveErrorView(request, response, status, model);
		return (modelAndView != null) ? modelAndView : new ModelAndView("error", model);
	}

    ...

}
```
* 스프링부트는 BasicErrorController를 자동으로 스프링 빈으로 등록한다. 이 컨트롤러는 만약 따로 경로를 지정하지 않으면 기본적으로 `/error`에 있는 오류페이지를 요청한다.
* 오류페이지 경로를 바꾸고 싶으면 application.properties에 server.error.path값을 바꾸면 된다.
* BasicErrorController 의 처리 순서
  1. 뷰 템플릿
     + resources/templates/error/500.html
     + resources/templates/error/5xx.html
     + 500.html처럼 더 구체적인 것이 우선순위가 높다.
  2. 정적 리소스
     + resources/static/error/400.html
     + resources/static/error/4xx.html
     + 뷰 템플릿과 마찬가지로 더 구체적인 것이 우선순위가 높다.
  3. 적용 대상이 없을 때
     + resources/templates/error.html
     + resources/static/error.html
     + 이름은 error.html이어야 한다.
## Exception Resolver
<center><img src="./../img/exception-resolver.png"></center>

```java
@Slf4j
public class MyHandlerExceptionResolver implements HandlerExceptionResolver {

    @Override
    public ModelAndView resolveException(HttpServletRequest request, HttpServletResponse response, Object handler, Exception ex) {
        
        log.info("Exception Resolver 실행");
        //return new ModelAndView();
        return null;
    }
}
```
* Exception Resolver는 예외 발생 시 실행된다.
* Exception Resolver에서 ModelAndView를 반환하면 해당 예외가 Exception Resolver에서 처리된 것으로 판단되어 WAS가 오류 처리요청을 다시 컨트롤러로 올리는 대신 정상응답을 내보낸다.
* Exception Resolver에서 null을 반환하면 다른 Exception Resolver를 찾고 만약 다른 Exception Resolver가 없으면 예외를 WAS까지 내려보낸다.
* resolveException에 예외를 처리하는 로직을 넣고 ModelAndVeiw를 반환하면 다시 오류를 처리하는 요청을 컨트롤러에 올려보내지 않고 뷰를 렌더링 하거나 정상응답을 내보내서 오버헤드를 줄일 수 있다.
* 스프링부트는 기본적으로 ExceptionHandlerExceptionResolver, ResponseStatusExceptionResolver, DefaultHandlerExceptionResolver를 제공한다. 리졸버가 적용되는 우선순위는 ExceptionHandlerExceptionResolver, ResponseStatusExceptionResolver, DefaultHandlerExceptionResolver이다.
* 다른 Exception Resolver를 WebMvcConfigurer를 상속하고 @Configuration가 지정된 클래스에 extendHandlerExceptionResolvers를 오버라이드 하여 추가할 수 있지만 잘 사용하지는 않는다. Exception Resolver는 HandlerExceptionResolver를 상속하여 만들 수 있다.
### ExceptionHandlerExceptionResolver
```java
@Getter
@Setter
@AllArgsConstructor
public class ErrorResult {
    private String code;
    private String message;
}
```
```java
@Slf4j
@RestController
public class ExceptionController {
    
    @ResponseStatus(HttpStatus.BAD_REQUEST)
    @ExceptionHandler(IllegalArgumentException.class)
    public ErrorResult illegalExHandler(IllegalArgumentException e) {
        log.error("[exceptionHandler] ", e);
        return new ErrorResult("Bad Request", e.getMessage());
    }

    @ExceptionHandler
    public ResponseEntity<ErrorResult> userExHandler(UserException e) {
        log.error("[exceptionHandler] ", e);
        ErrorResult errorResult = new ErrorResult("Bad Request", e.getMessage());
        return new ResponseEntity<>(errorResult, HttpStatus.BAD_REQUEST);
    }

    @ResponseStatus(HttpStatus.INTERNAL_SERVER_ERROR)
    @ExceptionHandler
    public ErrorResult exHandler(Exception e) {
        log.error("[exceptionHandler] ", e);
        return new ErrorResult("Internal Server Error", "내부 오류");
    }

    ...
    
}
```
* 예외가 발생하면 일반적으로 500코드를 내보내는데 이것을 바꿀 수 있다.
* 컨트롤러 내부에 @ExceptionHandler를 선언하고 처리하고 싶은 예외를 지정한 메서드를 정의하면 해당 컨트롤러에서 예외가 발생했을 때 ExceptionHandlerExceptionResolver가 @ExceptionHandler가 선언된 메서드를 찾고 리졸버에 들어온 예외를 해당 메서드가 처리할 수 있으면 그 메서드를 호출한다.
* @ExceptionHandler에 처리할 예외 타입을 지정할 수 있고, 배열의 형태로 여러개를 지정 가능한다.
* 배열의 형태로 처리할 예외타입을 여러개 지정했다면 파라미터에는 지정된 예외들의 부모타입이 와야한다.
* 더 구체적인(하위타입) 예외를 처리하는 메서드가 우선순위를 갖는다.
* @ExceptionHandler에 처리할 예외 타입을 지정하지 않으면 파라미터의 예외타입이 자동 지정된다.
* ExceptionHandler에서 예외가 처리된다면 오류처리 요청이 다시 컨트롤러로 올라가지 않고 바로 응답으로 내보내진다.
```java
// Target all Controllers annotated with @RestController
@ControllerAdvice(annotations = RestController.class)
public class ExampleAdvice1 {
    
    ...

}
// Target all Controllers within specific packages
@ControllerAdvice("org.example.controllers")
public class ExampleAdvice2 {

    ...

}
// Target all Controllers assignable to specific classes
@ControllerAdvice(assignableTypes = {ControllerInterface.class, AbstractController.class})
public class ExampleAdvice3 {
    
    ...

} 
```
* 각각의 컨트롤러에 @ExceptionHandler를 선언한 메서드를 정의할 필요없이 ControllerAdvice나 RestControllerAdvice에 이러한 메서드를 전부 정의할 수 있다.
* @ControllerAdvice 에 대상을 지정하지 않으면 모든 컨트롤러에 예외처리가 적용된다.
* @RestControllerAdvice는 @ControllerAdvice에 @ResponseBody 가 추가된 것과 같다.
* 특정 어노테이션이 지정된 컨트롤러에만 적용하거나,  특정 패키지에있는 컨트롤러에만 적용하거나, 특정 클래스(컨트롤러)를 지정하여 적용할 수 있다.
### ResponseStatusExceptionResolver
```java
@ResponseStatus(code = HttpStatus.BAD_REQUEST, reason = "잘못된 요청 오류")
public class BadRequestException extends RuntimeException {

    ...

}
```
```java
@GetMapping("/api/ex")
public String responseStatusEx() {
    throw new ResponseStatusException(HttpStatus.NOT_FOUND, "error.bad", new IllegalArgumentException());
}
```
* 예외가 발생하면 500코드를 내보내는데 이것을 바꿀 수 있다.
* @ResponseStatus는 내가 만든 예외에만 적용할 수 있다.
* 내가 직접 만들지 않은 예외는 ResponseStatusException을 throw하면 된다.
* 지정된 예외가 발생하면 바로 응답으로 내보내지는게 아니라 오류처리 요청이 컨트롤러까지 올라간다.
### DefaultHandlerExceptionResolver
```java
@Override
@Nullable
protected ModelAndView doResolveException(
        HttpServletRequest request, HttpServletResponse response, @Nullable Object handler, Exception ex) {

    try {
        if (ex instanceof HttpRequestMethodNotSupportedException) {
            return handleHttpRequestMethodNotSupported(
                    (HttpRequestMethodNotSupportedException) ex, request, response, handler);
        }
        else if (ex instanceof HttpMediaTypeNotSupportedException) {
            return handleHttpMediaTypeNotSupported(
                    (HttpMediaTypeNotSupportedException) ex, request, response, handler);
        }
        else if (ex instanceof HttpMediaTypeNotAcceptableException) {
            return handleHttpMediaTypeNotAcceptable(
                    (HttpMediaTypeNotAcceptableException) ex, request, response, handler);
        }

        ...

    }

    ...

}    
```
* DefaultHandlerExceptionResolver는 스프링 내부에서 발생하는 예외를 어떻게 처리할지가 정의되어있다.