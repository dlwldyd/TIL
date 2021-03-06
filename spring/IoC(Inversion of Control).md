# IoC(DI의 경우를 예로들어 설명)
## DI을 사용하지 않을 경우
```java
public class MemberService{
    //private MemberRepository memberRepository = new MemoryMemberRepository();
    private MemberRepository memberRepository = new JpaMemberRepository();
}
```
* 기존 프로그램은 클라이언트 구현 객체가 스스로 필요한 서버 구현 객체를 생성하고, 연결하고, 실행했다. 한마디로 구현 객체가 프로그램의 제어 흐름을 스스로 조종했다. 개발자 입장에서는 자연스러운 흐름이다.
## DI를 사용할 경우
```java
public class MemberService{
    private MemberRepository memberRepository;

    public MemberService(MemberRepository memberRepository){
        this.memberRepository = memberRepository;
    }
}
```
```java
@Configuration
public class SpringConfig {
    @Bean
    public MemberRepository memberRepository(){
        //return new MemoryMemberRepository();
        return new JpaMemberRepository();
    }
}
```
* 반면에 DI를 이용하면 구현 객체는 자신의 로직을 실행하는 역할만 담당한다. 프로그램의 제어 흐름은 이제 SpringConfig가 가져간다. MemberService는 필요한 인터페이스들을 호출하지만 어떤 구현 객체들이 실행될지 모른다. 
* __이렇듯 DI의 경우를 포함해서 프로그램의 제어 흐름을 직접 제어하는 것이 아니라 외부에서 관리하는 것을 제어의 역전(IoC)이라 한다.__
## 참고
### 프레임워크 vs 라이브러리
* 프레임워크가 내가 작성한 코드를 제어하고, 대신 실행하면 그것은 프레임워크가 맞다. ex) JUnit, spring
* 반면에 내가 작성한 코드가 직접 제어의 흐름을 담당한다면 그것은 프레임워크가 아니라 라이브러리다.