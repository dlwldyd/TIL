# OpenFeign
## FeignClient 사용
#### Order Service
```java
// order service의 핸들러 메서드
@GetMapping("/{username}/orders")
public ResponseEntity<List<ResponseOrder>> getOrders(@PathVariable String username) {
    ModelMapper modelMapper = new ModelMapper();
    List<Order> orders = orderService.getOrdersByUsername(username);
    List<ResponseOrder> responseOrders = orders.stream().map(order -> modelMapper.map(order, ResponseOrder.class)).collect(Collectors.toList());
    return ResponseEntity.ok(responseOrders);
}
```
#### User Service
```gradle
implementation 'org.springframework.cloud:spring-cloud-starter-openfeign'
```
```java
@FeignClient(name = "order-service", // FeignClient 이름, url이 없으면 FeignClient의 이름과 같은 이름의 어플레케이션에 요청을 보낸다.
            configuration = {FeignGlobalConfig.class, FeignConfig.class}) // config 파일 지정 가능
// @FeignClient(name = "test", url = "https://jsonplaceholder.typicode.com")
public interface OrderServiceClient {

    @GetMapping("/{userId}/orders")
    List<ResponseOrder> getOrders(@PathVariable(name = "userId") String userId);
}
```
```java
// feignClient 사용
List<ResponseOrder> orderList = orderServiceClient.getOrders(username);
```
 * @FeignClient 어노테이션에 적혀있는 어플리케이션에 `/{userId}/orders` 이라는 url로 Get 요청을 보내는 예제이다.
 * @FeignClient 어노테이션에 url을 명시하지 않으면 name에 적힌 서비스 이름으로 요청을 보내지만 url을 명시한다면 해당 url로 요청을 보낸다(외부 url 가능)
 * 언터페이스만 생성해서 @FeignClient만 붙이면 된다.
 * 기본적인 컨트롤러의 어노테이션과 똑같이 사용하면 된다. POST 요청을 보낼때도 body는 @RequestBody 어노테이션을 통해 보내면 된다.
 * 반환타입은 JSON, xml 등 의 포맷으로 온 응답을 파싱할 수 있으면 된다.
## 로그
```yml
# FeignClient를 사용하는 서비스의 application.yml 파일
# info 위로는 아무 로그 안나옴, debug 아래로 해야함
logging:
  level:
    com.example.userservice.client: DEBUG
```
```java
//feign.Logger 를 import;
@Bean
public Logger.Level feignLoggerLevel() {
    return Logger.Level.FULL;
}
```
* 로깅레벨을 debug 아래로 설정한다. 로그 레벨이 info 위로는 나오지 않기 때문에 아무 설정 하지 않으면 아무런 로그도 안나온다.
## 에러 처리
```java
@Component
public class FeignErrorDecoder implements ErrorDecoder {

    @Override
    public Exception decode(String methodKey, Response response) {
        switch (response.status()) {
            case 400:
                break;
            case 404:
                if (methodKey.contains("getOrders")) { // getOrders 메서드를 호출한 응답이 404일 경우
                    return new ResponseStatusException(HttpStatus.valueOf(response.status()), "User's orders is empty");
                }
                break;
            default:
                return new Exception(response.reason());
        }
        return null;
    }
}
```
```java
@SpringBootApplication
@EnableFeignClients // 붙여주지 않으면 open feign 작동 안함
public class FeignStudyApplication {

    public static void main(String[] args) {
        SpringApplication.run(FeignStudyApplication.class, args);
    }

}
```
* ErrorDecoder를 구현한 클래스를 스프링 빈으로 등록해주기만 하면 된다.
* methodKey는 FeignClient의 어떤 메서드에서 예외가 발생했는가이다.
* FeignClient의 메서드를 호출하는 쪽에서는 아무런 추가적인 코드를 쓰지 않아도 된다.
* @EnableFeignClients 붙이는거 필수
## 커스터마이징 한 ObjectMapper 등록
```java
@Configuration
public class FeignGlobalConfig {

    @Bean
    public Decoder feignDecoder() {
        MappingJackson2HttpMessageConverter jacksonConverter = new MappingJackson2HttpMessageConverter(customObjectMapper());
        ObjectFactory<HttpMessageConverters> objectFactory = () -> new HttpMessageConverters(jacksonConverter);
        return new ResponseEntityDecoder(new SpringDecoder(objectFactory, new ObjectProvider<>() {
            @Override
            public HttpMessageConverterCustomizer getObject(Object... args) throws BeansException {
                return null;
            }

            @Override
            public HttpMessageConverterCustomizer getIfAvailable() throws BeansException {
                return null;
            }

            @Override
            public HttpMessageConverterCustomizer getIfUnique() throws BeansException {
                return null;
            }

            @Override
            public HttpMessageConverterCustomizer getObject() throws BeansException {
                return null;
            }
        }));
    }

    private ObjectMapper customObjectMapper() {
        ObjectMapper objectMapper = new ObjectMapper();
        objectMapper.setPropertyNamingStrategy(PropertyNamingStrategies.SnakeCaseStrategy.INSTANCE);
        objectMapper.configure(DeserializationFeature.FAIL_ON_UNKNOWN_PROPERTIES, false);
        return objectMapper;
    }
}
```