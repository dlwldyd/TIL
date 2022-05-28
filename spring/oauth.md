# OAuth
## Workflow
<img src="./img/../../img/oauth.png">

1. 사용자가 웹사이트(클라이언트)에 OAuth 로그인을 시도한다.
2. 클라이언트는 OAuth 로그인페이지로 리다이렉트한다.
3. 사용자가 로그인페이지를 통해 로그인을 하면 authorization 서버는 사용자에게 authorization code를 발급한다.
4. 사용자는 발급받은 authorization code를 클라이언트에게 전달한다.
5. 클라이언트는 사용자로부터 받은 authorization code를 client id와 client secret과 함께 authorization 서버에 전달해 access token을 발급받는다.
6. 만약 사용자가 클라이언트에게 어떤 서비스를 요청하면 클라이언트는 발급받은 access token을 가지고 resource 서버의 api를 통해 해당 사용자의 권한으로 리소스에 접근할 수 있다.
## 구현
```yml
spring:
    security:
        oauth2:
        client:
            registration:
            google: //어느 리소스 서버인지
                clientId: 클라이언트 아이디
                clientSecret: 클라이언트 시크릿
                scope: # 클라이언트에게 허용된 리소스의 범위
                - email
                - profile
```
```java
@Configuration
@RequiredArgsConstructor
public class SecurityConfig extends WebSecurityConfigurerAdapter {

    private final PrincipalOauth2UserService principalOauth2UserService;

    @Value("${baseUrl}")
    private String loginSuccessUrl;

    @Override
    protected void configure(HttpSecurity http) throws Exception {
        http.authorizeRequests()
                .anyRequest().permitAll();
        http.headers()
                .frameOptions()
                .sameOrigin();
        http.oauth2Login() // 이걸 설정해야 /oauth2/authorization/google 로 요청을 받았을 때 구글로 로그인 하도록 리다이렉트함
                .loginPage("/login")
                .defaultSuccessUrl(loginSuccessUrl)
                .userInfoEndpoint() // 필수
                .userService(principalOauth2UserService); //파라미터로 들어가는 객체는 OAuth2UserService를 구현한 객체이다. 해당 객체에는 access 토큰을 받은 후 처리할 로직이 들어있다.
    }
}
```
```java
// 일반 폼 로그인에서는 UserDetails를 구현한 객체를 SecurityContextHolder에서 principal로 가지고있지만 OAuth를 사용하면 OAuth2User를 구현한 객체를 갖는다. 만약 OAuth와 폼 로그인 둘 다 사용하려면 둘다 구현하면 된다.
public class MemberDetails implements OAuth2User {

    private Member member;

    private Map<String, Object> attributes;

    public MemberDetails(Member member, Map<String, Object> attributes) {
        this.member = member;
        this.attributes = attributes;
    }

    // 유저 정보 리턴
    @Override
    public Map<String, Object> getAttributes() {
        return attributes;
    }

    //해당 유저의 권한을 리턴
    @Override
    public Collection<? extends GrantedAuthority> getAuthorities() {
        return null;
    }

    //sub값 리턴
    @Override
    public String getName() {
        return (String) attributes.get("sub");
    }

    public String getUsername() {
        return member.getUsername();
    }

    public String getNickname() {
        return member.getNickname();
    }

    public String getEmail() {
        return member.getEmail();
    }
}
```
```java
//액세스 토큰을 받기까지의 과정을 oauth2-client 가 전부 자동화해준다.
//여기에서는 authorization server 로 부터 받은 액세스 토큰과 유저 정보를 다룬다.
@Service
@RequiredArgsConstructor
@Transactional
@Slf4j
public class PrincipalOauth2UserService extends DefaultOAuth2UserService {

    private final PasswordEncoder passwordEncoder;
    private final MemberRepository memberRepository;

    //userRequest 에는 authorization server 로 부터 받은 유저정보 뿐만 아니라 어느 authorization server 로 부터 받았는지(getClientRegistration())에 대한 정보도 들어있다.
    //access token을 받은 후 로직을 작성할 수 있다.
    @Override
    public OAuth2User loadUser(OAuth2UserRequest userRequest) throws OAuth2AuthenticationException {

        OAuth2User oAuth2User = super.loadUser(userRequest);

        String provider = userRequest.getClientRegistration().getClientId();
        String providerId = oAuth2User.getAttribute("sub");
        String email = oAuth2User.getAttribute("email");
        String nickname = oAuth2User.getAttribute("given_name");
        String username = provider + "_" + providerId;
        String password = "ocj5df!983@5f";

        Optional<Member> byUsername = memberRepository.findByUsername(username);

        Member member;

        if (byUsername.isEmpty()) {
            member = new Member(username, password, email, nickname, passwordEncoder);
            memberRepository.save(member);
        } else {
            member = byUsername.get();
        }

        MemberDetails memberDetails = new MemberDetails(member, oAuth2User.getAttributes());

        return memberDetails;
    }
}
```
```java
// @AuthenticationPrincipal을 사용하면 SecurityContextHolder에 들어있는 principal을 가져올 수 있다. 이 때 해당 유저가 OAuth로 로그인 했으면 OAuth2User를 받고, 폼 로그인으로 로그인했으면 UserDetails를 받는다.
@GetMapping("/")
public String index(@AuthenticationPrincipal MemberDetails memberDetails) {

    ...

}
```
* 스프링은 spring oauth-client를 사용하면 위의 workflow의 1에서 5까지의 과정을 전부 자동화해준다. 나는 access token을 받은 후의 로직과 인증된 사용자에대한 로직만 짜면 된다.