# 메서드 방식 인증 처리
```java
@Configuration
//jsr250Enabled는 @RolesAllowed 어노테이션을 사용 가능하게함
@EnableGlobalMethodSecurity(securedEnabled = true, prePostEnabled = true, jsr250Enabled = true)
@RequiredArgsConstructor
@Order(1)
public class SecurityConfig extends WebSecurityConfigurerAdapter {

    ...

}
```
```java
@Controller
public class MethodSecurityExampleController {

    @GetMapping("/method")
    @PreAuthorize("hasRole('ROLE_USER') and #accountDto.username == principal.username") // springEL 넣을 수 있음
    // @Secured("ROLE_USER") -> 문자열(권한)로만 받을 수 있음, 스프링 시큐리티 어노테이션
    // @RolesAllowed("ROLE_USER") -> 문자열(권한)로만 받을 수 있음, 자바 어노테이션이고 기능은 @Secured와 같음
    public String methodSecurity(AccountDto accountDto, Model model, Principal principal) {

        model.addAttribute("method", "success");
        return "method";
    }
}
```
* 메서드 방식의 보안을 설정하기 위해서는 스프링 시큐리티 설정 클래스에 @EnableGlobalMethodSecurity(securedEnabled = true, prePostEnabled = true, jsr250Enabled = true)를 설정해줘야한다. securedEnabled, prePostEnabled, jsr250Enabled 속성은 각각 @Secured, @PreAuthorize, @RolesAllowed를 사용 가능하게 한다.
* @PreAuthorize는 SpringEL이 사용 가능하지만 나머지는 SpringEL 사용이 불가능하다. 만약 SpringEL을 사용할 필요가 없으면 @RolesAllowed나 @Secured를 사용하는 것이 좋다.