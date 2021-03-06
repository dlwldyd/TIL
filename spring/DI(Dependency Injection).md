# DI와 객체지항
## 객체 지향 프로그래밍
* 객체 지향 프로그래밍은 컴퓨터 프로그램을 명령어의 목록으로 보는 시각에서 벗어나 여러 개의 독립된 단위, 즉 "객체"들의 모임으로 파악하고자 하는 것이다.
* 객체 지향 프로그래밍은 다형성을 이용해 인터페이스와 구현체를 분리하여 컴퓨터 부품을 갈아 끼우듯이 컴포넌트를 쉽고 유연하게 변경하면서 개발할 수 있는 방법이다.
### 다형성의 본질
1. 인터페이스를 구현한 객체 인스턴스를 실행 시점에 유연하게 변경할 수 있다.
2. 클라이언트를 변경하지 않고, 서버의 구현 기능을 유연하게 변경할 수 있다.
### SOLID
로버트 마틴이 좋은 객체지향 설계의 5가지 원칙을 정리

1. SRP : 한 클래스는 하나의 책임만 가져야 한다. 변경이 있을 때 파급 효과가 적으면 단일 책임 원칙을 잘 따른 것이다.
2. OCP : 소프트웨어 요소는 확장에는 열려 있으나 변경에는 닫혀 있어야 한다. 다형성을 활용한다.
3. LSP : 프로그램의 객체는 프로그램의 정확성을 깨뜨리지 않으면서 하위 타입의 인스턴스로 바꿀 수 있어야 한다. 다형성에서 하위 클래스는 인터페이스 규약을 다 지켜야 한다는 것을 말한다.
4. ISP : 특정 클라이언트를 위한 인터페이스 여러 개가 범용 인터페이스 하나보다 낫다. 하나의 인터페이스를 여러 인터페이스로 분리하여 인터페이스가 명확해지고, 대체 가능성이 높아진다.
5. DIP : 구체화에 의존하지 말고 추상화에 의존해야 한다. 의존성 주입은 이 원칙을 따르는 방법 중 하나이다. 구현 클래스에 의존하지 말고, 인터페이스에 의존하라는 뜻이다.
## SRP, OCP, DIP의 위반
다형성만 가지고서는 좋은 객체지향 설계를 위해 SOLID를 지키며 프로그래밍 하기에는 매우 어렵다.
### DI을 사용하지 않을 경우
```java
public class MemberService{
    //private MemberRepository memberRepository = new MemoryMemberRepository();
    private MemberRepository memberRepository = new JpaMemberRepository();
}
```
* SRP 위반 : MemberService 객체는 서비스로직 뿐만 아니라 MemberRepository 구현 객체를 선택하는 역할까지 맡고있다.
* OCP 위반 : 리포지토리를 MemoryMemberRepository에서 JpaMemberRepository로 구현객체를 변경했을 뿐이지만 MemberService 객체가 구현클래스를 직접 선택하기 때문에 구현객체를 변경하려면 MemberService를 변경해야한다.
* DIP 위반 : MemberService 객체는 구현객체를 MemberRepository를 통해 받음으로써 인터페이스에 의존하지만, MemberService가 구현클래스를 직접 선택함으로써 구현클래스에도 동시에 의존한다.
* DI를 사용하지 않고 SOLID를 지키며 프로그래밍 할 수 있지만 코드가 매우 길고 복잡해진다.
### DI를 사용할 경우
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
* 스프링빈에 등록되는 구현클래스를 MemoryMemberRepository에서 JpaMemberRepository로 바꾸고 DI를 통해 MemberService가 구현객체를 받아 OCP와 DIP를 위반하지 않는다. 또한 MemberService 객체에서 MemberRepository 구현 객체를 선택하는 역할을 분리하여 SRP를 준수하였다.
* SpringConfig처럼 객체를 생성하고 관리하면서 의존관계를 연결해 주는 것을 `DI컨테이너`라 한다.
## 결론
__의존관계 주입(DI)이란 어떤 객체가 사용하는 구현객체를 직접 만들어 사용하는 것이 아니라 다른 객체로 부터 주입받아 사용함으로써 SOLID를 준수할 수 있고, 이를 통해 객체지향 프로그래밍의 장점을 극대화할 수 있도록 만들어준다.__