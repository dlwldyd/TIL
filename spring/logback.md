# Logback
## Appender
### Appender 종류 주로 사용하는거 5개
1. ConsoleAppender : 로그를 콘솔에 출력
2. FileAppender : 로그를 파일에 저장
3. RollingFileAppender : 여러개의 파일에 로그를 저장(로그 레벨마다 다른 파일에 로그를 저장할 수 있음)
4. SMTPAppender : 로그를 메일로 보냄
5. DBAppender : 로그를 데이터베이스에 저장
## 예시
```xml
<configuration scan=“true” scanPeriod=“30 seconds”> <!-- 30초마다 logback-spring.xml을 스캔해서 로딩함, 따라서 앱이 실행되는 중에 수정 가능 -->
…
</configuration>
```
* 가장 최상위
```xml
<property name=“type” value=“around_hub”/>
```
*  xml 내에서 변수 사용 가능
```xml
<appender name="FILE_DEBUG" class="ch.qos.logback.core.rolling.RollingFileAppender">
    <file>/var/tmp/applogs/loc-sdk-gw-sdb.log</file> <!-- 로그를 어느 파일에 저장할지 지정 -->
…
</appender>
```
* RollingFileAppender를 사용
```xml
<filter class="ch.qos.logback.classic.filter.ThresholdFilter">
    <level>ERROR</level>
</filter>
```
* ERROR 이상의 로그 레벨은 로깅하지 않음
```xml
<filter class="ch.qos.logback.classic.filter.ThresholdFilter">
    <level>ERROR</level>
    <onMatch>DENY</onMatch>
    <onMismatch>ACCEPT</onMismatch>
</filter>
```
* error level이면 로깅하지 않고 아니면 로깅함(error level만 로깅하지 않음)
```xml
<rollingPolicy class="ch.qos.logback.core.rolling.SizeAndTimeBasedRollingPolicy">
    <fileNamePattern>/var/tmp/applogs/loc-sdk-gw-sdb-%d{yyyy-MM-dd}.%i.log.gz</fileNamePattern> <!-- 파일 이름 형식 지정, 압축 형식 지정 -->
    <maxFileSize>100MB</maxFileSize> <!-- 최대 파일 사이즈 100MB -->
    <maxHistory>10</maxHistory> <!-- 10일 동안 저장 -->
    <totalSizeCap>10GB</totalSizeCap> <!-- ??? -->
</rollingPolicy>
```
* RolliingFileAppender를 사용할 때만 넣음
```xml
<encoder>
    <pattern>%d{HH:mm:ss.SSS} [%thread] %-5level %logger{36} - %msg%n</pattern>
</encoder>
```
* 로그의 출력 형태를 지정
```xml
<root level="INFO"> <!-- INFO 레벨 이상의 로그 -->
    <appender-ref ref="CONSOLE"/> <!-- appender name=“CONSOLE” 가 appender로 사용됨 -->
</root>
```
* 로그 전역 설정, 모든 패키지에 대한 로그 설정
```xml
<logger name="com.kakaomobility.location" level="DEBUG" additivity="false"> <!-- com.kakaomobility.location 패키지에 있는 로그만 적용됨, 상위 appender를 상속받을 지 안받을 지(겹쳤을 때 중복으로 출력할지 말지, false면 중복출력X) -->
    <appender-ref ref="CONSOLE_DEBUG”/>
</logger>
```
* 로그 지역 설정, 특정 패키지에 대한 로그를 설정함
```xml
<springProfile name="dev">
…
</springProfile>
```
* 프로필별로 로깅 설정 가능
```xml
<encoder class="net.logstash.logback.encoder.LogstashEncoder">
    <includeCallerData>true</includeCallerData>
    <fieldNames class="net.logstash.logback.fieldnames.ShortenedFieldNames"> <!-- json 에 들어가는 필드 중 특정 필드를 무시할 수 있음 -->
        <version>[ignore]</version>
        <thread>[ignore]</thread>
        <levelValue>[ignore]</levelValue>
    </fieldNames>
</encoder>
```
* 로그를 json 형태로 출력
## logback-spring.xml 실제 사용 예
```xml
<?xml version="1.0" encoding="UTF-8" ?>
<configuration>
    <conversionRule conversionWord="clr" converterClass="org.springframework.boot.logging.logback.ColorConverter"/>
    <appender name="STDOUT" class="ch.qos.logback.core.ConsoleAppender">
        <encoder>
            <pattern>%green(%d{HH:mm:ss.SSS}) %magenta([%thread]) %clr(%-5level) %cyan(%logger{36}) - %msg%n</pattern>
        </encoder>
    </appender>
    <appender name="CONSOLE" class="ch.qos.logback.core.ConsoleAppender">
        <filter class="ch.qos.logback.classic.filter.LevelFilter">
            <level>INFO</level>
        </filter>
        <encoder class="net.logstash.logback.encoder.LogstashEncoder">
            <includeCallerData>true</includeCallerData>
            <fieldNames class="net.logstash.logback.fieldnames.ShortenedFieldNames">
                <version>[ignore]</version>
                <thread>[ignore]</thread>
                <levelValue>[ignore]</levelValue>
            </fieldNames>
        </encoder>
    </appender>
    <springProfile name="local">
        <logger name="com.kakaomobility.location" level="DEBUG" additivity="false">
            <appender-ref ref="STDOUT"/>
        </logger>
        <logger name="com.kakaomobility.location" level="INFO" additivity="false">
            <appender-ref ref="CONSOLE"/>
        </logger>
        <logger name="com.kakaomobility.location.logging.RequestResponseLoggingFilter" level="INFO" additivity="false">
            <appender-ref ref="CONSOLE"/>
        </logger>
        <logger name="com.kakaomobility.location.advice.ExceptionHandlerAdvice" level="ERROR" additivity="false">
            <appender-ref ref="CONSOLE"/>
        </logger>
        <root level="INFO">
            <appender-ref ref="STDOUT"/>
        </root>
    </springProfile>
    <springProfile name="dev">
        <logger name="com.kakaomobility.location" level="INFO" additivity="false">
            <appender-ref ref="CONSOLE"/>
        </logger>
        <logger name="com.kakaomobility.location.logging.RequestResponseLoggingFilter" level="INFO" additivity="false">
            <appender-ref ref="CONSOLE"/>
        </logger>
        <logger name="com.kakaomobility.location.advice.ExceptionHandlerAdvice" level="ERROR" additivity="false">
            <appender-ref ref="CONSOLE"/>
        </logger>
        <root level="INFO">
            <appender-ref ref="CONSOLE"/>
        </root>
    </springProfile>
    <springProfile name="sbx">
        <logger name="com.kakaomobility.location" level="INFO" additivity="false">
            <appender-ref ref="CONSOLE"/>
        </logger>
        <logger name="com.kakaomobility.location.logging.RequestResponseLoggingFilter" level="INFO" additivity="false">
            <appender-ref ref="CONSOLE"/>
        </logger>
        <logger name="com.kakaomobility.location.advice.ExceptionHandlerAdvice" level="ERROR" additivity="false">
            <appender-ref ref="CONSOLE"/>
        </logger>
        <root level="INFO">
            <appender-ref ref="CONSOLE"/>
        </root>
    </springProfile>
    <springProfile name="cbt">
        <logger name="com.kakaomobility.location" level="INFO" additivity="false">
            <appender-ref ref="CONSOLE"/>
        </logger>
        <logger name="com.kakaomobility.location.logging.RequestResponseLoggingFilter" level="INFO" additivity="false">
            <appender-ref ref="CONSOLE"/>
        </logger>
        <logger name="com.kakaomobility.location.advice.ExceptionHandlerAdvice" level="ERROR" additivity="false">
            <appender-ref ref="CONSOLE"/>
        </logger>
        <root level="INFO">
            <appender-ref ref="CONSOLE"/>
        </root>
    </springProfile>
    <springProfile name="prod">
        <logger name="com.kakaomobility.location" level="INFO" additivity="false">
            <appender-ref ref="CONSOLE"/>
        </logger>
        <logger name="com.kakaomobility.location.logging.RequestResponseLoggingFilter" level="INFO" additivity="false">
            <appender-ref ref="CONSOLE"/>
        </logger>
        <logger name="com.kakaomobility.location.advice.ExceptionHandlerAdvice" level="ERROR" additivity="false">
            <appender-ref ref="CONSOLE"/>
        </logger>
        <root level="INFO">
            <appender-ref ref="CONSOLE"/>
        </root>
    </springProfile>
</configuration>
```
## pattern
* %logger{length} : Logger name
* %-5level : 로그 레벨, -5는 출력의 고정폭 값(무조건 5글자 크기)
* %msg : 로그 메세지 영역(==%message)
* ${PID:-} : 프로세스 id
* %d : 로그 기록 시간
* %p : 로깅 레벨
* %F : 로깅이 발생한 프로그램 파일명
* %M : 로깅이 발생한 메서드의 이름
* %I : 로깅이 발생한 호출지의 정보
* %L : 로깅이 발생한 호출지의 라인 넘버
* %thread : 현재 Thread 명
* %t : 로깅이 발생한 Thread 명
* %c : 로깅이 발생한 카테고리
* %C : 로깅이 발생한 클래스 명
* %m : 로그 메세지
* %n : 줄바꿈
* %% : %출력
* %r : 어플리케이션 실행 후 로깅이 발생한 시점까지의 시간
## json 형태로 로그 남기기
```java
@Override
public void doFilter(ServletRequest request, ServletResponse response, FilterChain chain) throws IOException, ServletException {
    // import org.slf4j.MDC;
    MDC.put("traceId", generateTraceId()); // traceId는 ThreadLocal별로 고유의 값을 할당해 주는 것이 좋다.
    
    final Map<String, Object> markerMap = new HashMap<>();
    final String method = requestWrapper.getMethod();
    final String path = requestWrapper.getRequestURI();
    
    markerMap.put("method", method);
    markerMap.put("path", path);

    ...

    log.info(Markers.appendEntries(markerMap), "{} {} request done", method, path);
}

private String generateTraceId() {
    // 일반적인 난수 발생은 전역적으로 난수를 생성하기 때문에 만약 다른 쓰레드에서 같은 시간에 난수를 생성하게 되면 같은 값을 가질 수 있다. 하지만 ThreadLocalRandom을 사용해 난수를 발생시키면 멀티쓰레드 환경에서도 안전하게 난수를 생성 가능하다.
    return String.format("%s-%010d", hostName, Math.abs(ThreadLocalRandom.current().nextLong() % 89999999999L + 10000000000L));
}
```
* Slf4j.MDC는 멀티 클라이언트 환경에서 각각의 클라이언트에 다른 값을 부여하여 클라이언트를 구별해 로깅할 수 있도록 한다.
* LogstashEncoder를 사용하여 로깅을 하면 로그가 자동으로 json형태가 되는데 Markers를 사용하여 json에 원하는 필드를 추가할 수 있다.