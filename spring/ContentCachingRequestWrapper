# HttpServletRequest의 Body 로깅하기
## ContentCachingRequestWrapper
HttpServletRequest의 Body는 InputStream으로 들어가 있기 때문에 한번 값을 조회하면 다시 조회할 수 없다. 따라서 로깅을 하기 위해 body를 조회하면 controller 단에서 해당 값을 읽을 수 없게 된다는 문제가 발생한다. 이러한 문제를 해결하기 위하여 ContentCachingRequestWrapper를 사용하는데, 필터 단에서 로깅을 위해 inputstream을 읽어들일 때 해당 값을 캐시에 저장한 뒤 다음에 해당 body를 읽을 때는 캐시에서 읽어들이는 방법을 통해 http body 값을 계속해서 읽을 수 있게 된다. 단, ContentCachingRequestWrapper는 SpringMvc에서만 사용 가능하다. 따라서 Webflux를 사용한다면 Decorator 패턴을 사용해 body 값을 캐시에 저장하도록 직접 개발할 필요가 있다.
## 사용 예시
```java
@Slf4j
public class RequestResponseLoggingFilter implements Filter {

    private final static int CACHE_LIMIT = 20480;

    private final String hostName;

    private final ObjectMapper objectMapper;

    public RequestResponseLoggingFilter() {
        this.hostName = getHostName();
        this.objectMapper = new ObjectMapper();
    }

    @Override
    public void init(FilterConfig filterConfig) throws ServletException {
        Filter.super.init(filterConfig);
    }

    @Override
    public void doFilter(ServletRequest request, ServletResponse response, FilterChain chain) throws IOException, ServletException {

        final long startTime = System.currentTimeMillis();

        ContentCachingRequestWrapper requestWrapper = new ContentCachingRequestWrapper((HttpServletRequest) request, CACHE_LIMIT);
        ContentCachingResponseWrapper responseWrapper = new ContentCachingResponseWrapper((HttpServletResponse) response);
        MDC.put("traceId", generateTraceId());

        chain.doFilter(requestWrapper, responseWrapper);

        final Map<String, Object> markerMap = new HashMap<>();
        final String method = requestWrapper.getMethod();
        final String path = requestWrapper.getRequestURI();
        final long elapsedTimeMillis = startTime - System.currentTimeMillis();

        markerMap.put("method", method);
        markerMap.put("path", path);
        markerMap.put("elapsed", elapsedTimeMillis);
        markerMap.put("responseStatus", responseWrapper.getStatus());

        String requestBody = new String(requestWrapper.getContentAsByteArray());
        if (!requestBody.isEmpty()) {
            try {
                Map<String, Object> requestJsonMap = objectMapper.readValue(requestBody, new TypeReference<>() {
                });
                markerMap.put("requestBody", requestJsonMap);
            } catch (MismatchedInputException | JsonParseException e) {
                try {
                    List<Object> requestJsonList = objectMapper.readValue(requestBody, new TypeReference<>() {
                    });
                    markerMap.put("requestBody", requestJsonList);
                } catch (MismatchedInputException | JsonParseException ex) {
                    markerMap.put("requestBody", requestBody);
                }
            }
        }

        String responseBody = new String(responseWrapper.getContentAsByteArray());
        if (!responseBody.isEmpty()) {
            try {
                Map<String, Object> responseJsonMap = objectMapper.readValue(responseBody, new TypeReference<>() {
                });
                markerMap.put("responseBody", responseJsonMap);
            } catch (MismatchedInputException | JsonParseException e) {
                try {
                    List<Object> responseJsonList = objectMapper.readValue(responseBody, new TypeReference<>() {
                    });
                    markerMap.put("responseBody", responseJsonList);
                } catch (MismatchedInputException | JsonParseException ex) {
                    markerMap.put("responseBody", responseBody);
                }
            }
        }
        log.info(Markers.appendEntries(markerMap), "{} {} request done in {} ms.", method, path, elapsedTimeMillis);
        responseWrapper.copyBodyToResponse(); // 필수, 이거 없으면 response를 여러번 읽을 수 없음
    }

    @Override
    public void destroy() {
        Filter.super.destroy();
    }

    private String generateTraceId() {
        return String.format("%s-%010d", hostName, Math.abs(ThreadLocalRandom.current().nextLong() % 89999999999L + 10000000000L));
    }

    private String getHostName() {
        String name = System.getenv("HOSTNAME");
        if (name != null) {
            return name;
        }
        String lineStr;
        try {
            Process process = Runtime.getRuntime().exec("hostname");
            BufferedReader br = new BufferedReader(new InputStreamReader(process.getInputStream()));
            while ((lineStr = br.readLine()) != null ) {
                name = lineStr;
            }
        } catch (Exception e) {
            throw new RuntimeException("hostname get failed.");
        }
        return name;
    }
}
```
