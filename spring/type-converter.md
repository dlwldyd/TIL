# 타입 컨버터
## 컨버터
```java
@Getter
@EqualsAndHashCode
@AllArgsConstructor
public class IpPort {
    private String ip;
    private int port;
}
```
```java
@Slf4j
public class StringToIpPortConverter implements Converter<String, IpPort> {

    @Override
    public IpPort convert(String source) {
        //"127.0.0.1:8080" -> IpPort("127.0.0.1", 8080)
        log.info("convert source={}", source);
        String[] split = source.split(":");
        String ip = split[0];
        int port = Integer.parseInt(split[1]);
        return new IpPort(ip, port);
    }
}
```
```java
@Configuration
public class WebConfig implements WebMvcConfigurer {

    @Override
    public void addFormatters(FormatterRegistry registry) {
        registry.addConverter(new StringToIpPortConverter());
    }
}
```
* org.springframework.core.convert.converter패키지의 컨버터 인터페이스를 상속받아 컨버터를 구현할 수 있다.
* WebMvcConfigurer를 상속한 클래스의 addFormatter함수를 오버라이드하여 스프링은 내부에서 사용하는 ConversionService 에 컨버터를 추가할 수 있다.
* 컨버터를 추가하면 추가한 컨버터가 기본 컨버터 보다 높은 우선순위를 갖는다.
## 포맷터
```java
@Slf4j
public class MyNumberFormatter implements Formatter<Number> {

    @Override
    public Number parse(String text, Locale locale) throws ParseException {
        log.info("text={}, locale={}", text, locale);
        //"1,000" -> 1000
        NumberFormat format = NumberFormat.getInstance(locale);
        return format.parse(text);
    }

    @Override
    public String print(Number object, Locale locale) {
        log.info("object={}, locale={}", object, locale);
        //1000 -> "1,000"
        NumberFormat instance = NumberFormat.getInstance(locale);
        return instance.format(object);
    }
}
```
```java
@Configuration
public class WebConfig implements WebMvcConfigurer {

    @Override
    public void addFormatters(FormatterRegistry registry) {
        registry.addFormatter(new MyNumberFormatter());
    }
}
```
```java
@Data
public class Form {
    @NumberFormat(pattern = "###,###")
    private Integer number;
    
    @DateTimeFormat(pattern = "yyyy-MM-dd HH:mm:ss")
    private LocalDateTime localDateTime;
}
```
* 포맷터는 문자를 다른 타입으로, 다른 타입을 문자로 변환하는 컨버터를 포맷터라 한다.
* Formatter 인터페이스를 상속받아 포맷터를 구현할 수 있다.
* WebMvcConfigurer를 상속한 클래스의 addFormatter함수를 오버라이드하여 스프링은 내부에서 사용하는 ConversionService 에 포맷터를 추가할 수 있다.
* 같은 기능의 컨버터와 포맷터가 있으면 컨버터가 우선순위가 더 높다.
* 스프링은 어노테이션을 기반으로 하는 숫자관련 포맷터와 날짜관련 포맷터를 기본으로 제공한다. 이 어노테이션을 통해 원하는 포맷을 지정해서 타입을 변환할 수 있다.
  + @NumberFormat : 숫자 관련 형식 지정 포맷터 사용
  + @DateTimeFormat : 날짜 관련 형식 지정 포맷터 사용
## 타임리프
```html
<ul>
    <li>${ipPort}: <span th:text="${ipPort}" ></span></li>
    <li>${{ipPort}}: <span th:text="${{ipPort}}" ></span></li>
</ul>
```
```html
<form th:object="${form}" th:method="post">
    th:field <input type="text" th:field="*{ipPort}"><br/>
    th:value <input type="text" th:value="*{ipPort}"><br/>
    <input type="submit"/>
</form>
```
* ${{...}}를 사용하면 컨버전서비스를 이용해 뷰를 렌더링할 때 컨버팅 된 결과를 출력해준다.
* th:field를 사용하면 value로 컨버전서비스를 이용해 컨버팅된 결과를 출력해준다.