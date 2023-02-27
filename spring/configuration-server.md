# Configuration Server
## Configuration Server 생성
<img src="../img/yml-files.png"/>

* 하나의 디렉토리에 각 마이크로서비스들이 사용할 프로파일별 yml 파일들을 모아둔다.
* 해당 디렉토리를 git을 사용해 관리한다. 단, 원격 리포지토리에 관리할지는 선택
```java
@SpringBootApplication
@EnableConfigServer // 필수
public class ConfigServiceApplication {

	public static void main(String[] args) {
		SpringApplication.run(ConfigServiceApplication.class, args);
	}

}
```
* 프로젝트를 생성할 때 의존성에 Spring Cloud Config의 Config Server를 추가한다.
* 프로젝트를 생성하면 메인함수가 있는 클래스에 @EnableConfigServer를 추가해준다.
```yml
# ecommerce.yml 파일
# file://C:\spring_cloud\git-local-repo 하위에 존재한다.
# 깃으로 관리되고있어야 한다.(리모트 리포지토리로는 관리될 필요X)
token:
  expiration_time: 86400000
  secret: FOJ2@#FJ33TF@#5Ffom#!@3@kkf2#$2FF234f2#gmFOJ2@#FJ33TF@#5Ffom#!@3@kkf2#$2FF234f2#gm-default

gateway:
  ip: 192.168.0.8
```
```yml
server:
  port: 8888

spring:
  application:
    name: config-server
  cloud:
    config:
      server:
        git:
          default-label: master # 깃허브 default 브랜치 이름이 main이 아닐 때 적어주자
          uri: file://C:\spring_cloud\git-local-repo
```
<img src="../img/config-server-result.png"/>

* Config Server의 yml 파일에 다른 마이크로서비스들이 사용할 yml 파일들이 있는 디렉토리 경로를 명시해준다.
* localhost:8888/{yml file name}/{profile name} 으로 어떤 파일이 불러와지는지 알 수 있다.
* 만약 존재하지 않는 yml 파일이면 위 사진에서 propertySources 이후의 값이 없는 채로 오고, 존재하지 않는 profile이면 default 프로파일이 온다.
## 마이크로서비스와 Config Server 연동
```gradle
// spring cloud config server를 사용하기 위한 dependency
implementation 'org.springframework.cloud:spring-cloud-starter-config'
implementation 'org.springframework.cloud:spring-cloud-starter-bootstrap'
```
* 마이크로서비스와 config server를 연동하기 위해서는 위와 같은 의존성을 추가해줘야한다.
```yml
# bootstrap.yml 파일 추가
spring:
  cloud:
    config:
      uri: http://localhost:8888 # config server 주소
      name: ecommerce # 가지고올 yml 파일 이름
```
* bootstrap.yml 파일을 추가해 위처럼 어디에 있는 config server에서 어떤 설정 파일을 가져올지 명시한다.
<img src="../img/yml-fetch.png"/>

* 마이크로서비스를 실행하면 위의 사진처럼 설정파일을 가져오는 로그가 남는 것을 볼 수 있다.
```yml
# userservice의 application.yml 파일

...

#token:
#  expiration-time: 86400000
#  secret: FOJ2@#FJ33TF@#5Ffom#!@3@kkf2#$2FF234f2#gmFOJ2@#FJ33TF@#5Ffom#!@3@kkf2#$2FF234f2#gm
```
```java
@GetMapping("/health_check")
public String status() {
    return String.format("It's Working in User Service, " +
            "port(local.server.port) = %s<br>" +
            "port(server.port) = %s<br>" +
            "token = %s<br>" +
            "token expiration time = %s",
            env.getProperty("local.server.port"),
            env.getProperty("server.port"),
            env.getProperty("token.secret"),
            env.getProperty("token.expiration_time"));
}
```
<img src="../img/health-check-result.png"/>

* 실제로 마이크로서비스의 yml파일에는 token정보를 주석처리해서 없애줬지만 config server에서 해당 정보를 가져오기 때문에 userservice에서 토큰 정보에 접근할 수 있다.
## Config Server에서 설정 값 수정
config server에서 설정값을 수정하면 userservice를 재기동해야 한다. 만약 재기동을 하기 싫으면 2가지 방법이 있는데 첫 번째 방법은 Actuator의 refresh endpoint를 사용하는 방법이 있고 두 번째 방법은 spring cloud bus를 사용하는 방법이 있다.
### Actuator Refresh
```gradle
// actuator 사용을 위한 dependency
implementation 'org.springframework.boot:spring-boot-starter-actuator'
```
* actuator 기능을 하용하려면 위 사진과 같이 의존성을 추가해줘야 한다. actuator 기능자체는 spring boot에 포함되어있다.
```yml
# userservice의 application.yml 파일

# 사용할 actuator의 endpoint를 yml파일에 등록해야 사용할 수 있다.
management:
  endpoints:
    web:
      exposure:
        include: refresh, health, beans
```
* userservice의 application.yml 파일에 사용하고 싶은 actuator의 endpoint를 명시한다.
```java
http.authorizeHttpRequests()
    .antMatchers("/actuator/**").permitAll();
```
* 만약 spring security를 사용하고 있다면 actuator를 사용하기 위한 url을 허용하자
<img src="../img/actuator-health.png"/>

---
<img src="../img/actuator-beans.png"/>

* actuator를 통해 서비스에 대한 모니터링 정보를 위의 사진처럼 json 형태로 받아볼 수 있다.
#### 기존의 결과 값
<img src="../img/health-check-result.png"/>

---
#### 수정 후
```yml
# ecommerce.yml 파일
token:
  expiration_time: 86400000
  secret: fix

gateway:
  ip: 192.168.0.8
```
* userservice가 가지고 오는 설정파일 값을 수정한다.
<img src="../img/actuator-refresh.png"/>

---
* post 방식으로 userservice에 /actuator/refresh 요청을 보내면 userservice는 다시 설정파일에서 값을 가져온다.
<img src="../img/ecommerce-fix.png"/>

---
* 설정 값이 바뀐 것을 확인할 수 있다.
## 프로파일 나누기
```yml
# ecommerce.yml
token:
  expiration_time: 86400000
  secret: FOJ2@#FJ33TF@#5Ffom#!@3@kkf2#$2FF234f2#gmFOJ2@#FJ33TF@#5Ffom#!@3@kkf2#$2FF234f2#gm-default

gateway:
  ip: 192.168.0.8
```
```yml
# ecommerce-dev.yml
token:
  expiration_time: 86400000
  secret: FOJ2@#FJ33TF@#5Ffom#!@3@kkf2#$2FF234f2#gmFOJ2@#FJ33TF@#5Ffom#!@3@kkf2#$2FF234f2#gm-dev

gateway:
  ip: 192.168.0.8
```
```yml
# ecommerce-prod.yml
token:
  expiration_time: 86400000
  secret: FOJ2@#FJ33TF@#5Ffom#!@3@kkf2#$2FF234f2#gmFOJ2@#FJ33TF@#5Ffom#!@3@kkf2#$2FF234f2#gm-prod

gateway:
  ip: 192.168.0.8
```
* config 파일을 {yml 파일 이름}-{프로파일 이름}.yml 로 여러개로 나눈다.
```yml
# user-service의 bootstrap.yml 파일
spring:
  cloud:
    config:
      uri: http://localhost:8888 # config server 주소
      name: ecommerce # 가지고올 yml 파일 이름
  profiles:
    active: dev
```
```yml
# apigateway-service의 bootstrap.yml 파일
spring:
  cloud:
    config:
      uri: http://localhost:8888 # config server 주소
      name: ecommerce # 가지고올 yml 파일 이름
  profiles:
    active: dev
```
* 위와 같이 프로파일을 설정하면 config 파일을 땡길 때 ecommerce-dev.yml 파일을 땡겨온다.
```yml
# user-service의 bootstrap.yml 파일
spring:
  cloud:
    config:
      uri: http://localhost:8888 # config server 주소
      name: ecommerce # 가지고올 yml 파일 이름
  profiles:
    active: prod
```
```yml
# apigateway-service의 bootstrap.yml 파일
spring:
  cloud:
    config:
      uri: http://localhost:8888 # config server 주소
      name: ecommerce # 가지고올 yml 파일 이름
  profiles:
    active: prod
```
* 위와 같이 프로파일을 설정하면 config 파일을 땡길 때 ecommerce-prod.yml 파일을 땡겨온다.