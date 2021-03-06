# 컴포넌트 스캔을 통한 DI
컴포넌트 스캔에서 의존관계 주입 방법은 4가지 방법이 있다.
* 생성자 주입
* setter 주입
* 필드 주입
* 일반 메서드 주입
## 생성자 주입
* 생성자를 통해서 의존관계를 주입 받는 방법이다.
* 생성자 호출시점에 딱 1번만 호출되는 것이 보장된다.
```java
@Component
public class MemberServiceImpl implements MemberService {
    private final MemberRepository memberRepository;

    //생성자가 하나만 있으면 @Autowired를 생략해도 된다.
    @Autowired
    public MemberServiceImpl(MemberRepository memberRepository) {
        this.memberRepository = memberRepository;
    }
}
```
## setter 주입
* setter라 불리는 필드의 값을 변경하는 수정자 메서드를 통해서 의존관계를 주입하는 방법이다.
* 선택, 변경 가능성이 있는 의존관계에 사용한다.
```java
@Component
public class MemberServiceImpl implements MemberService {
    private MemberRepository memberRepository;
    
    //@Autowired는 기본적으로 주입할 대상이 없으면 오류가 난다. 주입할 대상이 없어도 동작하게 하려면 @Autowired(required = false)로 지정하면 된다.
    @Autowired(required = false)
    public setMemberRepository(MemberRepository memberRepository){
        this. memberRepository = memberRepository;
    }
}
```
## 필드 주입
* 외부에서 해당 필드의 변경이 불가능해서 테스트 하기 힘들다는 치명적인 단점이 있다. 자바 테스트 코드에는 당연히 @Autowired가 동작하지 않는다. @SpringBootTest 처럼 스프링 컨테이너를 테스트에 통합한 경우에만 가능하다.
* 애플리케이션의 실제 코드와 관계 없는 테스트 코드, 스프링 설정을 목적으로 하는 @Configuration 같은 곳에서만 특별한 용도로 사용해야한다.
```java
@Component
public class MemberServiceImpl implements MemberService {
    @Autowired private MemberRepository memberRepository;
}
```
## 일반 메서드 주입
* 한번에 여러 필드를 주입 받을 수 있다.
* 일반적으로 잘 사용하지 않는다.
```java
@Component
public class MemberServiceImpl implements MemberService {
    private MemberRepository memberRepository;

    @Autowired
    public void init(MemberRepository memberRepository){
        this.memberRepository = memberRepository;
    }
}
```
## 의존관계 주입 방법 선택 시 주의점(생성자 주입 방식 선택 권장)
* 대부분의 의존관계 주입은 한번 일어나면 애플리케이션 종료시점까지 의존관계를 변경할 일이 없다. 생성자 주입은 객체를 생성할 때 딱 1번만 호출되므로 이후에 호출되는 일이 없다. 따라서 생정자 주입 방식을 선택하면 불변하게 설계할 수 있다.
* setter 주입을 사용하면 setXxx 메서드를 public으로 열어두어야 한다. 누군가 실수로 변경할 수 도 있고, 변경하면 안되는 메서드를 열어두는 것은 좋은 설계 방법이 아니다. 메서드 주입 또한 마찬가지이다.
* 생성자 주입을 사용하면 필드에 final 키워드를 사용할 수 있다. 그래서 생성자에서 혹시라도 값이 설정되지 않는 오류를 컴파일 시점에 막아준다.
* 필드 주입의 경우에는 외부에서 값을 변경할 수 없다는 약점이 있기 때문에 테스트 코드 같은 한정적인 경우에만 사용하는 것이 좋다.
* __기본으로 생성자 주입을 사용하고, 필수 값이 아닌 경우에는 수정자 주입이나 메서드 주입 방식을 옵션으로 부여하면 된다. 생성자 주입과 setter 또는 메서드 주입을 동시에 사용할 수 있다.__