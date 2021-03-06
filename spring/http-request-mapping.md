# HTTP 요청 매핑
## 매핑 어노테이션
### 컨트롤러 등록
```java
@RequestMapping
@ResponseBody //RestController
public interface OrderControllerV1 {

    @GetMapping("/v1/request")
    //인터페이스에서는 어노테이션이 매핑되는 이름(여기서는 itemId)가 있어야 한다.
    String request(@RequestParam("itemId") String itemId);

    @GetMapping("/v1/no-log")
    String noLog();
}
```
* 컨트롤러를 @Bean 으로 수동 등록할 때 @Controller가 아니라 @RequestMapping을 써줘야 컨트롤러로 인식한다.(@Controller를 쓰면 수동 등록하기 전에 컴포넌트 스캔으로 자동 등록된다.)
### 여러개의 URI 매핑
```java
@ResponseBody
@RequestMapping(value = {"/hello-basic", "/hello-go"})
public String helloBasic(){
    log.info("helloBasic");
    return "ok";
}
```
* 배열로 여러개의 URI를 넣을 수 있다.
### 특정 HTTP 메서드 요구
```java
@ResponseBody
@RequestMapping(value = "/mapping-get", method = RequestMethod.GET)
public String mappingGet() {
    log.info("mappingGet");
    return "ok";
}
```
* 특정 HTTP 메서드 요청만 허용
```java
@ResponseBody
@GetMapping(value = "/mapping-get")
public String mappingGet() {
    log.info("mapping-get");
    return "ok";
}
```
* HTTP 메서드를 지정할 때 축약 어노테이션을 사용할 수 있다.
* @GetMapping, @PostMapping, @PutMapping, @DeleteMapping, @PatchMapping 등
## 쿼리 파라미터 매핑
### @PathVariable
```java
@ResponseBody
@GetMapping("/mapping/{userId}")
public String mappingPath(@PathVariable("userId") String data){
    log.info("mappingPath userId={}", data);
    return "ok";
}
```
* 변수명과 path가 같으면 생략 가능(@PathVariable("userId") String userId -> @PathVariable userId)
* 여러개의 @PathVariable을 사용해도 됨
```java
@ResponseBody
@RequestMapping("/request-param-required")
public String RequestParamRequired(
        @RequestParam(required = false) String username,
        @RequestParam(required = false) Integer age){
    log.info("username = {}, age = {}", username, age);
    return "ok";
}
```
* required=false를 세팅하면 해당 파라미터가 없어도 실행된다.
* default는 required=true로 세팅되어 있다.
```java
@ResponseBody
@RequestMapping("/request-param-default")
public String RequestParamDefault(
        @RequestParam(defaultValue = "guest") String username,
        @RequestParam(defaultValue = "-1") int age){
    log.info("username = {}, age = {}", username, age);
    return "ok";
}
```
* defaultValue 설정을 통해 default값 설정 가능
* 빈 문자가 들어올 경우 default 값으로 실행된다.
```java
@ResponseBody
@RequestMapping("/request-param-map")
public String RequestParamMap(@RequestParam MultiValueMap<String, Object> paramMap){

    log.info("username = {}, age = {}", paramMap.get("username"), paramMap.get("age"));
    return "ok";
}
```
* 모든 파라미터를 MultiValueMap을 이용해서 받을 수 있다.
### @ModelAttribute
```java
@ResponseBody
@RequestMapping("/model-attribute")
public String ModelAttribute(@ModelAttribute HelloData helloData) {

    log.info("helloData = {}", helloData);

    return "ok";
}
```
* helloData의 로컬변수 이름과 파라미터의 이름이 같아야 하고 setter가 있어야한다.
* @ModelAttribute를 통해 받은 데이터는 자동으로 Model에 담기게 된다. key는 클래스 이름의 맨앞글자를 소문자로 바꾼 것이다.
* @ModelAttribute는 생략 가능하다. 하지만 @ModelAttribute를 생략하면 Model에 해당 객체는 담기지 않는다.
```java
@ModelAttribute("helloData")
public HelloData hellodata() {
    return new HelloData();
}
```
* @ModelAttribute는 컨트롤러에 있는 별도의 메서드에 적용할 수 있다. 이렇게하면 해당 컨트롤러를 요청할 때 helloData 에서 반환한 값이 자동으로 모델(model)에 담기게 된다. 시스템 성능을 향상시키기 위해 이렇게 사용하지 않고 각각의 컨트롤러 메서드에서 모델에 직접 데이터를 담아서 처리해도 된다.
### RequestAttributes
```java
@PostMapping("/add")
public String addItemV6(Item item, RedirectAttributes redirectAttributes) {
    Item savedItem = itemRepository.save(item);
    redirectAttributes.addAttribute("itemId", savedItem.getId());
    redirectAttributes.addFlashAttribute("itemName", savedItem.getItemName());
    redirectAttributes.addAttribute("status", true);
    return "redirect:/basic/items/{itemId}";
}
```
* 스프링에서 리다이렉트에서 데이터를 전달할 때 사용하는 방법이다.
* 리다이렉트 후 get방식으로 넘어올 때 pathVariable과 쿼리 파라미터까지 addAttribute로 넘길 수 있다.
* addAttribute로 넘긴 데이터중 pathVariable로 받지 않는 데이터는 모두 쿼리 파라미터로 넘어온다.
* 보안상 URL에 보여지면 안되거나 새로고침했을 때 데이터가 넘어오지 않기를 원한다면 addFlashAttribute로 넘길 수 있다.
### 특정 파라미터값 요구
```java
@ResponseBody
@GetMapping(value = "/mapping-param", params = "mode=debug")
public String mappingParam() {
    log.info("mappingParam");
    return "ok";
}
```
* params="mode" -> 파라미터 이름만 넣어도됨
* params="!mode" -> 특정 파라미터가 없어야 호출되도록 할 수 있음
* params="mode=debug" -> 파라미터의 값을 요구할 수 있음
* params="mode!=debug" (! = ) -> 파라미터의 값이 특정 값이 아님을 요구할 수 있음
* params = {"mode=debug","data=good"} -> 여러개도 지정 가능
## 헤더 매핑
### 특정 헤더 요구
```java
@ResponseBody
@GetMapping(value = "/mapping-header", headers = "mode=debug")
public String mappingHeader() {
    log.info("mappingHeader");
    return "ok";
}
```
* headers="mode", -> 특정 헤더 필드를 요구할 수 있음
* headers="!mode" -> 특정 헤더 필드가 없어야 하는 것을 요구할 수 있음
* headers="mode=debug" -> 특정 헤더 필드의 특정 값을 요구 할 수 있음
* headers="mode!=debug" (! = ) -> 특정 헤더 필드가 특정 값을 가지지 않도록 요구 할 수 있음
### Content-type
```java
@ResponseBody
@PostMapping(value = "/mapping-consumes", consumes = "application/json")
public String mappingConsumes() {
    log.info("mappingConsumes");
    return "ok";
}
```
* Content-Type 의 경우 headers 를 쓰면 안됨
* consumes="application/json" -> Content-Type 이 특정 값을 가지면 호출하도록 할 수 있음
* MediaType.APPLICATION_JSON_VALUE -> Content-Type 헤더 기반 추가 매핑 Media Type
* consumes="!application/json" -> 특정 content type을 가지지 않도록 지정할 수 있음
* consumes="application/*"
* consumes="*\/\*" -> 와일드 카드 문자 쓸 수 있음
* consumes={"text/plain", "application/*"} -> 여러개 지정 가능
### Accept
```java
@ResponseBody
@PostMapping(value = "/mapping-produce", produces = "text/html")
public String mappingProducesHtml() {
    log.info("mappingProducesHtml");
    return "ok html";
}
@ResponseBody
@PostMapping(value = "/mapping-produce", produces = "application/json")
public String mappingProducesJson() {
    log.info("mappingProducesJson");
    return "ok json";
}
```
* produces = "text/html" -> 특정 Accept 값 요구
* produces = "!text/html" -> 특정 Accept 값이 아님을 요구
* produces = "text/*"
* produces = "*\/\*" -> 와일드 카드 문자 사용 가능
* produces = {"text/plain", "application/*"} -> 여러개 지정 가능
* produces = "text/plain;charset=UTF-8" -> 문자 인코딩 방식 지정 가능
* produces = MediaType.TEXT_PLAIN_VALUE -> 미디어 타입으로 지정 가능
* Accept 값이 여러개 오면 q 값이 높은 쪽이 mapping 됨
## 바디 매핑
### HttpEntity
```java
@PostMapping("/request-body-string")
public HttpEntity<String> requestBodyString(HttpEntity<String> httpEntity) throws IOException {
    String messageBody = httpEntity.getBody();

    log.info("messageBody = {}", messageBody);

    return new HttpEntity<>("ok");
}
```
* HttpEntity 는 http 의 헤더와 바디 정보를 편리하게 조회할 수 있게 해주는 객체이다.
* httpEntity.getHeaders(); 로 헤더 정보도 조회할 수 있다.
* httpEntity.getBody(); 로 바디 정보를 조회할 수 있다.
* HttpEntity 를 상속받는 RequestEntity 와 ResponseEntity 를 사용할 수도 있다.
* InputStream 을 문자열로 바꾸려면 제네릭을 String(HttpEntity\<String\>)으로 설정한다. 다른 타입도 가능하다.
### @ResponseBody
```java
@ResponseBody
@PostMapping("/request-body-string")
public String requestBodyString(@RequestBody String messageBody) {

    log.info("messageBody = {}", messageBody);

    return "ok";
}
```
* @ResponseBody를 쓰면 리턴한 문자열을 그대로 바디에 넣는다.
### @RequestBody
```java
@ResponseBody
@PostMapping("/request-body-json")
public String requestBodyJson(@RequestBody HelloData helloData)  {

    log.info("username = {}, age = {}", helloData.getUsername(), helloData.getAge());

    return "ok";
}
```
```java
@ResponseBody
@PostMapping("/request-body-json")
public String requestBodyJson(HttpEntity<HelloData> httpEntity)  {
    HelloData helloData = httpEntity.getBody();

    log.info("username = {}, age = {}", helloData.getUsername(), helloData.getAge());

    return "ok";
}
```
```java
@ResponseBody
@PostMapping("/request-body-json")
public HelloData requestBodyJson(@RequestBody HelloData helloData)  {

    log.info("username = {}, age = {}", helloData.getUsername(), helloData.getAge());

    return helloData;
}
```
* HTTP 요청의 바디가 json 형식으로 올 때 사용한다.
* 로컬 변수 이름이 json 키 값과 같아야한다.
* HttpEntity<HelloData>를 사용해도 된다.
* 리턴값을 객체로 반환하면 타입 컨버터가 바디에 json형식으로 값을 넣어 응답한다.(json(요청) -> HelloData(함수내) -> json(응답))