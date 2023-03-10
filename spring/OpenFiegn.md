# OpenFeign
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
@FeignClient(name = "order-service") // 서비스 이름
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
 * 언터페이스만 생성해서 @FeignClient만 붙이면 된다.
 * 기본적인 컨트롤러의 어노테이션과 똑같이 사용하면 된다. POST 요청을 보낼때도 body는 @RequestBody 어노테이션을 통해 보내면 된다.
 * 반환타입은 JSON, xml 등 의 포맷으로 온 응답을 파싱할 수 있으면 된다.