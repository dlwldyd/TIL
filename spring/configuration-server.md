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
server:
  port: 8888

spring:
  application:
    name: config-server
  cloud:
    config:
      server:
        git:
          uri: file://C:\spring_cloud\git-local-repo
```
<img src="../img/config-server-result.png"/>

* Config Server의 yml 파일에 다른 마이크로서비스들이 사용할 yml 파일들이 있는 디렉토리 경로를 명시해준다.
* localhost:8888/{yml file name}/{profile name} 으로 어떤 파일이 불러와지는지 알 수 있다.
* 만약 존재하지 않는 yml 파일이면 위 사진에서 propertySources 이후의 값이 없는 채로 오고, 존재하지 않는 profile이면 default 프로파일이 온다.