# DocumentContext
## TypeRef<> 사용
```java
String response = myFeignClient.getResponse();
ObjectMapper objectMapper = new ObjectMapper();
objectMapper.setPropertyNamingStrategy(PropertyNamingStrategies.SnakeCaseStrategy.INSTANCE);
objectMapper.configure(DeserializationFeature.FAIL_ON_UNKNOWN_PROPERTIES, false);
Configuration configuration = Configuration.builder().mappingProvider(new JacksonMappingProvider(objectMapper)).build();
DocumentContext documentContext = JsonPath.using(configuration).parse(response);
List<DataEntity> dataEntityList = documentContext.read("$['result']", new TypeRef<>() {});
```
* DocumentContext에서 TypeRef<>를 사용하고 싶으면 Configuration에 ObjectMapper를 추가하여 사용해야함
```
{
    "result": [
        {
            "timestamp": "2023-07-25 14:59:47.833000",
            "price": 49.99
        },
        {
            "timestamp": "2023-07-25 14:59:48.333000",
            "price": 31.25
        }
    ]
}
```
## jayway jsonpath
* `$`: 루트 노드로 모든 Path 표현식은 이 기호로 시작된다. ex) $['result'] -> 루트 노드에 있는 result를 나타냄
* `@`: 처리되고 있는 현재 노드를 나타내고 필터 조건자에서 사용된다.
* `*`: 와일드카드로 모든 요소와 매칭이 된다 ex) $['result'][*] -> result 내의 모든 요소
* `.`: Dot 표현식의 자식노드
* `['field']`: 해당 필드만 매칭(리스트와 매칭되면 리스트 내의 요소가 아니라 리스트 자체를 나타냄)
* `[?(<expression>)]`: 필터 표현식으로 필터 조건자가 참인 경우에 매칭되는 모든 요소를 만을 처리한다 ex) $['result'][*].[?(@.price == 49.99)] -> price가 49.99인 데이터만 반환