# 메세지 컨버터, Argument Resolver
## 메시지 컨버터
뷰 템플릿으로 HTML을 생성해서 응답하는 것이 아니라, HTTP API처럼 JSON 데이터를 HTTP 메시지 바디에서 직접 읽거나 쓰는 경우 알맞은 메세지 컨버터를 찾아(Content-type 헤더, 클래스 타입에 영향을 받는다.) 호출하여 메세지를 변환해주고(변환은 Argument Resolver가 한다.) 응답의 경우에도 알맞은 메세지 컨버터를 찾아서(Accept 헤더, 리턴 타입에 영향을 받는다) 메세지를 변환하여(변환은 ReturnValue Handler가 한다) 메세지 바디에 써준다.
<center><img src="./../img/message-converter.png"></center>

### 요청
```java
HandlerAdapter ha = getHandlerAdapter(mappedHandler.getHandler());
```
```java
mv = ha.handle(processedRequest, response, mappedHandler.getHandler());
```
1. Dispatcher Servlet에서 핸들러를 실행할 수 있는 핸들러 어댑터를 찾고 handle 함수 실행
```java
@Override
@Nullable
public final ModelAndView handle(HttpServletRequest request, HttpServletResponse response, Object handler)
        throws Exception {

    return handleInternal(request, response, (HandlerMethod) handler);
}
```
```java
@Override
protected ModelAndView handleInternal(HttpServletRequest request,
        HttpServletResponse response, HandlerMethod handlerMethod) throws Exception {

    ModelAndView mav;
    checkRequest(request);

    // Execute invokeHandlerMethod in synchronized block if required.
    if (this.synchronizeOnSession) {
        HttpSession session = request.getSession(false);
        if (session != null) {
            Object mutex = WebUtils.getSessionMutex(session);
            synchronized (mutex) {
                mav = invokeHandlerMethod(request, response, handlerMethod);
            }
        }
        else {
            // No HttpSession available -> no mutex necessary
            mav = invokeHandlerMethod(request, response, handlerMethod);
        }
    }
    else {
        // No synchronization on session demanded at all...
        mav = invokeHandlerMethod(request, response, handlerMethod);
    }

    ...

    return mav;
}
```
```java
@Nullable
protected ModelAndView invokeHandlerMethod(HttpServletRequest request,
        HttpServletResponse response, HandlerMethod handlerMethod) throws Exception {

    ServletWebRequest webRequest = new ServletWebRequest(request, response);
    try {
        
        ...

        ServletInvocableHandlerMethod invocableMethod = createInvocableHandlerMethod(handlerMethod);

        ...

        invocableMethod.invokeAndHandle(webRequest, mavContainer);
        
        ...

    }
    finally {
        webRequest.requestCompleted();
    }
}
```
2. invokeHandlerMethod 함수에서 ServletInvocableHandlerMethod 객체 생성 후 invokeAndHandle 함수 실행
```java
public void invokeAndHandle(ServletWebRequest webRequest, ModelAndViewContainer mavContainer,
			Object... providedArgs) throws Exception {

    Object returnValue = invokeForRequest(webRequest, mavContainer, providedArgs);
    
    ...

}
```
```java
@Nullable
public Object invokeForRequest(NativeWebRequest request, @Nullable ModelAndViewContainer mavContainer,
        Object... providedArgs) throws Exception {

    Object[] args = getMethodArgumentValues(request, mavContainer, providedArgs);
    if (logger.isTraceEnabled()) {
        logger.trace("Arguments: " + Arrays.toString(args));
    }
    return doInvoke(args);
}
```
```java
protected Object[] getMethodArgumentValues(NativeWebRequest request, @Nullable ModelAndViewContainer mavContainer,
			Object... providedArgs) throws Exception {

    MethodParameter[] parameters = getMethodParameters();
    if (ObjectUtils.isEmpty(parameters)) {
        return EMPTY_ARGS;
    }

    Object[] args = new Object[parameters.length];
    for (int i = 0; i < parameters.length; i++) {
        MethodParameter parameter = parameters[i];
        parameter.initParameterNameDiscovery(this.parameterNameDiscoverer);
        args[i] = findProvidedArgument(parameter, providedArgs);
        if (args[i] != null) {
            continue;
        }
        if (!this.resolvers.supportsParameter(parameter)) {//Argument Resolver가 해당 파라미터를 지원하는지 체크
            throw new IllegalStateException(formatArgumentError(parameter, "No suitable resolver"));
        }
        try {
            args[i] = this.resolvers.resolveArgument(parameter, mavContainer, request, this.dataBinderFactory);//실제 컨트롤러(핸들러)에 전달할 객체 생성
        }
        catch (Exception ex) {
            // Leave stack trace for later, exception may actually be resolved and handled...
            if (logger.isDebugEnabled()) {
                String exMsg = ex.getMessage();
                if (exMsg != null && !exMsg.contains(parameter.getExecutable().toGenericString())) {
                    logger.debug(formatArgumentError(parameter, exMsg));
                }
            }
            throw ex;
        }
    }
    return args;
}
```
```java
//HandlerMethodArgumentResolverComposite 구현체
//이 함수가 true면 resolver를 찾는 것을 그만두고 resolveArugument 함수 실행, false면 다른 resolver를 찾는다.
@Override
public boolean supportsParameter(MethodParameter parameter) {
    return getArgumentResolver(parameter) != null;
}
```
```java
//HandlerMethodArgumentResolverComposite 구현체
//여기서 return되는 값을 바인딩한다.
//HandlerMethodArgumentResolverComposite는 여러개의 Argument Resolver를 모아놓은 것
@Override
@Nullable
public Object resolveArgument(MethodParameter parameter, @Nullable ModelAndViewContainer mavContainer,
        NativeWebRequest webRequest, @Nullable WebDataBinderFactory binderFactory) throws Exception {

    HandlerMethodArgumentResolver resolver = getArgumentResolver(parameter);
    if (resolver == null) {
        throw new IllegalArgumentException("Unsupported parameter type [" +
                parameter.getParameterType().getName() + "]. supportsParameter should be called first.");
    }
    return resolver.resolveArgument(parameter, mavContainer, webRequest, binderFactory);
}
```
```java
//RequestResponseBodyMethodProcessor 구현체
@Override
public Object resolveArgument(MethodParameter parameter, @Nullable ModelAndViewContainer mavContainer,
        NativeWebRequest webRequest, @Nullable WebDataBinderFactory binderFactory) throws Exception {

    parameter = parameter.nestedIfOptional();
    Object arg = readWithMessageConverters(webRequest, 
    
    ...
    
}
```
3. getMethodArgumentValues 함수에서 supportsParameter를 호출하여 해당 파라미터를 지원하는지 체크하고 지원하면 resolveArgument를 호출하여 실제 컨트롤러(핸들러)에 넘어갈 객체 생성. resolveArgument를 호출했을 때 메세지 컨버터를 호출하여 메시지를 읽을 수 있는지, 메시지를 쓸 수 있는지 판단한다.
```java
@Nullable
protected <T> Object readWithMessageConverters(HttpInputMessage inputMessage, MethodParameter parameter,
        Type targetType) throws IOException, HttpMediaTypeNotSupportedException, HttpMessageNotReadableException {

    ...
    
    try {
        message = new EmptyBodyCheckingHttpInputMessage(inputMessage);

        for (HttpMessageConverter<?> converter : this.messageConverters) {
            Class<HttpMessageConverter<?>> converterType = (Class<HttpMessageConverter<?>>) converter.getClass();
            GenericHttpMessageConverter<?> genericConverter =
                    (converter instanceof GenericHttpMessageConverter ? (GenericHttpMessageConverter<?>) converter : null);
            if (genericConverter != null ? genericConverter.canRead(targetType, contextClass, contentType) :
                    (targetClass != null && converter.canRead(targetClass, contentType))) {
                if (message.hasBody()) {
                    HttpInputMessage msgToUse =
                            getAdvice().beforeBodyRead(message, parameter, targetType, converterType);
                    body = (genericConverter != null ? genericConverter.read(targetType, contextClass, msgToUse) :
                            ((HttpMessageConverter<T>) converter).read(targetClass, msgToUse));
                    body = getAdvice().afterBodyRead(body, msgToUse, parameter, targetType, converterType);
                }
                else {
                    body = getAdvice().handleEmptyBody(null, message, parameter, targetType, converterType);
                }
                break;
            }
        }
    }
    catch (IOException ex) {
        throw new HttpMessageNotReadableException("I/O error while reading input message", ex, inputMessage);
    }

    ...

    return body;
}
```
```java
public interface HttpMessageConverter<T> {

	/**
	 * Indicates whether the given class can be read by this converter.
	 * @param clazz the class to test for readability
	 * @param mediaType the media type to read (can be {@code null} if not specified);
	 * typically the value of a {@code Content-Type} header.
	 * @return {@code true} if readable; {@code false} otherwise
	 */
	boolean canRead(Class<?> clazz, @Nullable MediaType mediaType);

	/**
	 * Indicates whether the given class can be written by this converter.
	 * @param clazz the class to test for writability
	 * @param mediaType the media type to write (can be {@code null} if not specified);
	 * typically the value of an {@code Accept} header.
	 * @return {@code true} if writable; {@code false} otherwise
	 */
	boolean canWrite(Class<?> clazz, @Nullable MediaType mediaType);

	/**
	 * Return the list of media types supported by this converter. The list may
	 * not apply to every possible target element type and calls to this method
	 * should typically be guarded via {@link #canWrite(Class, MediaType)
	 * canWrite(clazz, null}. The list may also exclude MIME types supported
	 * only for a specific class. Alternatively, use
	 * {@link #getSupportedMediaTypes(Class)} for a more precise list.
	 * @return the list of supported media types
	 */
	List<MediaType> getSupportedMediaTypes();

	/**
	 * Return the list of media types supported by this converter for the given
	 * class. The list may differ from {@link #getSupportedMediaTypes()} if the
	 * converter does not support the given Class or if it supports it only for
	 * a subset of media types.
	 * @param clazz the type of class to check
	 * @return the list of media types supported for the given class
	 * @since 5.3.4
	 */
	default List<MediaType> getSupportedMediaTypes(Class<?> clazz) {
		return (canRead(clazz, null) || canWrite(clazz, null) ?
				getSupportedMediaTypes() : Collections.emptyList());
	}

	/**
	 * Read an object of the given type from the given input message, and returns it.
	 * @param clazz the type of object to return. This type must have previously been passed to the
	 * {@link #canRead canRead} method of this interface, which must have returned {@code true}.
	 * @param inputMessage the HTTP input message to read from
	 * @return the converted object
	 * @throws IOException in case of I/O errors
	 * @throws HttpMessageNotReadableException in case of conversion errors
	 */
	T read(Class<? extends T> clazz, HttpInputMessage inputMessage)
			throws IOException, HttpMessageNotReadableException;

	/**
	 * Write an given object to the given output message.
	 * @param t the object to write to the output message. The type of this object must have previously been
	 * passed to the {@link #canWrite canWrite} method of this interface, which must have returned {@code true}.
	 * @param contentType the content type to use when writing. May be {@code null} to indicate that the
	 * default content type of the converter must be used. If not {@code null}, this media type must have
	 * previously been passed to the {@link #canWrite canWrite} method of this interface, which must have
	 * returned {@code true}.
	 * @param outputMessage the message to write to
	 * @throws IOException in case of I/O errors
	 * @throws HttpMessageNotWritableException in case of conversion errors
	 */
	void write(T t, @Nullable MediaType contentType, HttpOutputMessage outputMessage)
			throws IOException, HttpMessageNotWritableException;

}
```
4. readWithMessageConverters함수를 실행하면서 메세지 컨버터를 통해 Content-Type과 Accept의 미디어 타입을 보고 해당 메시지를 변환할 수 있는지 판단하게 된다.(Argument Resolver는 파라미터에 바인딩할 수 있는지 판단)
### 응답
HTTP 응답의 경우에도 위와 마찬가지의 방식으로 메시지 컨버터를 통해 객체를 메시지로 변환한다.
## Argument Resolver 확장
```java
@Target(ElementType.PARAMETER)
@Retention(RetentionPolicy.RUNTIME)
public @interface Login {
}
```
```java
@GetMapping("/")
public String homeLoginV3ArgumentResolver(@Login Member loginMember, Model 
model) {

    ...

}
```
* Argument Resolver를 이용하면 직접 만든 커스텀 어노테이션을 이용해서 파라미터를 바인딩 할 수 있다.
```java
@Slf4j
public class LoginMemberArgumentResolver implements HandlerMethodArgumentResolver {

    @Override
    public boolean supportsParameter(MethodParameter parameter) {
        log.info("supportsParameter 실행");

        boolean hasLoginAnnotation = parameter.hasParameterAnnotation(Login.class);
        boolean hasMemberType = Member.class.isAssignableFrom(parameter.getParameterType());

        return hasLoginAnnotation && hasMemberType;
    }

    @Override
    public Object resolveArgument(MethodParameter parameter, ModelAndViewContainer mavContainer,
                                  NativeWebRequest webRequest, WebDataBinderFactory binderFactory) throws Exception {

        log.info("resolveArgument 실행");

        HttpServletRequest request = (HttpServletRequest) webRequest.getNativeRequest();
        HttpSession session = request.getSession(false);
        if (session == null) {
            return null;
        }

        return session.getAttribute(SessionConst.LOGIN_MEMBER);
    }
}
```
* supportsParameter(MethodParameter parameter) : 컨트롤러 메서드의 파라미터에 어노테이션이 있으면 List\<HandlerMethodArgumentResolver> resolvers의 supportsParameter를 순서대로 실행하면서 파라미터를 바인딩 할 수 있는 Argument Resolver를 찾는다. 이 함수의 리턴값이 false라면 다음 Argument Resolver로 넘어간다. true면 Argument Resolver를 찾는 것을 그만두고 resolveArgument함수를 실행한다.
* resolveArgument() : 컨트롤러 호출 직전에 호출 되어서 필요한 파라미터 정보를 생성해준다. 여기서 반환되는 값이 파라미터에 바인딩 된다.
```java
@Configuration
public class WebConfig implements WebMvcConfigurer {
    @Override
    public void addArgumentResolvers(List<HandlerMethodArgumentResolver> resolvers) {
        resolvers.add(new TestArgumentResolver());
        resolvers.add(new LoginMemberArgumentResolver());
    }
}
```
* Argument Resolver를 추가하려면 WebMvcConfigurer 인터페이스를 상속받고 addArgumentResolvers를 오버라이드 해야한다.
* resolvers.add()함수를 통해 내가 만든 ArgumentResolver를 추가할 수 있다.