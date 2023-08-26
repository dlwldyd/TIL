# WebClient
```java
MultipartBodyBuilder builder = new MultipartBodyBuilder();
builder.part("type", type);
builder.part("file", filePart).filename(fileName);

WebClient.create()
    .post()
    .uri(URI.create(url))
    .contentType(MediaType.MULTIPART_FORM_DATA)
    .body(BodyInserters.fromMultipartData(builder.build())) // multipart로 body 추가
    .exchangeToMono(response -> { // 응답을 받고난 뒤의 로직
        Optional<MediaType> mediaType = response.headers().contentType();
        return response.bodyToMono(String.class) // 응답의 body는 String 타입으로 반환
                .map(s -> MyResponse.builder()
                        .statusCode(response.statusCode())
                        .mediaType(mediaType)
                        .body(s)
                        .build());
    })
    .retryWhen(Retry.backoff(2, Duration.ofMillis(100)) // 요청 실패시 재요청 : Retry.fixedDelay(n, t) t초마다 n번 요청, Retry.backoff(n, t) n번 요청하는데 시간간격 t초가 지수함수적으로 증가
            .filter(throwable -> throwable instanceof WebClientRequestException) // WebClientRequestException 발생 시에만 retry함
            .onRetryExhaustedThrow((retryBackoffSpec, retrySignal) -> { // retry 횟수를 채웠는데도 제대로 응답을 못 받았을 때 로직
                throw new FileUploadFailureException(ErrorCode.ERROR__UPLOAD_FAILED, ServiceCode.SVC_Tenth, "premature timeout");
            }));
```
```java
WebClient.create()
    .put()
    .uri(URI.create(url))
    .contentType(MediaType.TEXT_PLAIN)
    .headers(httpHeaders -> { // header 추가
        httpHeaders.add("header1", header1);
        httpHeaders.add("header2", header2);
        httpHeaders.add("header3", header3);
    })
    .body(BodyInserters.fromValue(content)) // text plain으로 body 추가
    .exchangeToMono(response -> handleResponse(response, path))
    .retryWhen(Retry.backoff(2, Duration.ofMillis(100))
            .filter(throwable -> throwable instanceof WebClientRequestException).onRetryExhaustedThrow((retryBackoffSpec, retrySignal) -> {
                throw new FileUploadFailureException(ErrorCode.ERROR__UPLOAD_FAILED, ServiceCode.SVC_Tenth, "premature timeout");
            }));
```
* 위의 메서드만으로는 http요청이 안감, 리턴 값으로 받은 Mono를 subscribe 해야함
* 컨트롤러단에서 리턴값을 Mono로 주면 굳이 subscribe 안해도 내부적으로 해줌
```java
String uri = "http://localhost:8080"
Mono<String> webClient = execute(uri);
webClient.subscribe(); // 이걸 해야 실제로 http 요청이 들어감

public Mono<String> execute(String uri) {
    WebClient.create()
    .get()
    .uri(URI.create(uri))
    .exchangeToMono(response -> handle(response));
}
```