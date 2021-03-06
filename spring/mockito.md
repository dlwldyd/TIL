# Mockito
## stubbing
```java
@ExtendWith(MockitoExtension.class)
class ItemServiceTest {

    @InjectMocks
    private ItemService itemService;

    @Mock
    private ItemRepository itemRepository;

    ...

}
```
* Mockito를 사용하기 위해서는 테스트 객체에 @ExtendWith(MockitoExtension.class)를 붙여줘야한다.
* @Mock : 해당 어노테이션이 붙은 객체를 가짜 객체로 대체한다.
* @InjectMocks : 해당 어노테이션이 붙은 객체를 생성할 때 mock 객체를 주입한다.
## 메서드 반환값 정의, verify()
```java
Item item = new Item();

//목객체의 메서드에 대한 리턴 값을 정의한다.
when(itemRepository.save(any(Item.class))).thenReturn(item);
doReturn(item).when(itemRepository).save(any(Item.class));

//메서드 호출 시 예외를 던지도록 할 수도 있다.
when(itemRepository.save(any())).thenThrow(new RuntimeException());

//any()면 어떤객체가 파라미터로 들어가든 item을 반환한다.
when(itemRepository.save(any())).thenReturn(item);
//any(Item.class)면 Item 타입의 객체가 파라미터로 들어가면 item을 반환하다.
when(itemRepository.save(any(Item.class))).thenReturn(item);

//특정 값이 파라미터로 들어갔을 때의 반환 값을 정의한다.
when(itemRepository.findById(1L)).thenReturn(item);

//반환 타입이 void 타입일 때는 doNothing()을 활용하자
doNothing().when(itemRepository).deleteById(any(Long.class));

//mock 객체의 메서드가 호출 됐는지를 검증한다.
verify(itemRepository).save();
//mock 객체의 메서드가 정해진 횟수만큼 호출 됐는지를 검증한다.
verify(itemRepository, times(2)).save();
//mock 객체의 메서드가 정해진 횟수 이상으로 호출 됐는지를 검증한다.
verify(itemRepository, atLeast(2)).save(any(Item.class));
//mock 객체의 메서드가 정해진 횟수 이하으로 호출 됐는지를 검증한다.
verify(itemRepository, atMost(2)).save(any(Item.class));
//mock 객체의 메서드가 호출 되지 않았는지를 검증한다.
verify(itemRepository, never()).save(any(Item.class));
```
* mock 객체를 사용하기 위해서는 mock 객체의 메서드를 호출했을 때 어떤 값을 반환할지를 정의해야한다. 이 때 정의하는 메서드는 테스트가 실행되는 동안 호출되는 메서드만 정의하면 된다.(내가 직접 호출하는 것 말고도 다른 메서드에 의해 간접적으로 호출 되는 메서드까지 전부)
## @MockBean, @SpyBean
```java
@ExtendWith(MockitoExtension.class)
@SpringBootTest
class ItemServiceTest {

    @Autowired
    private ItemService itemService;

    @Autowired
    private MemberService memberService;

    @MockBean(name = "itemRepository")
    private ItemRepository itemRepository;

    @SpyBean(name = "memberRepository")
    private MemberRepository memberRepository;

    @Test
    void test(){
        Item item = new Item();
        Member member = new Member();
        
        when(itemRepository.save(any(Item.class))).thenReturn(item);
        when(memberRepository.save(any(Member.class))).thenReturn(member);

        //메서드에 대한 반환값을 정의하였으면 @MockBean이든 @SpyBean이든 둘다 동작한다.
        itemService.saveItem(item);
        memberService.saveMember(member);

        //@MockBean의 경우 메서드에 대한 반환값을 정의하지 않으면 해당 메서드를 호출할 수 없지만 @SpyBean의 실제 메서드를 호출한다.
        //itemService.findById(1L);
        memberService.findById(1L);
    }
}
```
* @MockBean과 @SpyBean을 사용하면 통합테스트에서 의존성 주입을 할 때 실제 스프링 빈 객체가 주입되는 대신에 가짜 객체가 주입된다.
* @MockBean의 경우 메서드에 대한 반환값을 정의하지 않았으면 정의하지 않은 메서드는 사용할 수 없지만 @SpyBean의 경우에는 실제 객체의 메서드가 호출된다.
## static 메서드 mocking하기
```gradle
testImplementation 'org.mockito:mockito-inline:4.4.0'
```
```java
@Mock
private SecurityContext securityContext;

@Test
void test(){
    MockedStatic<SecurityContextHolder> mSecurityContextHolder = mockStatic(SecurityContextHolder.class);

    when(SecurityContextHolder.getContext()).thenReturn(securityContext);

    mSecurityContextHolder.close();
}
```
* static 메서드를 mocking 하기 위해서는 mockito-inline에 대한 의존성을 추가해줘야 한다. 스프링 부트에 기본적으로 들어가는 mockito는 mockito-core이다.
* `mockStatic(mocking할 static 메서드가 있는 클래스)`를 통해 static 메서드에 대한 mocking을 할 수 있다. 그리고 해당 static 메서드를 다 사용한 후 close()메서드를 호출해 주는 것이 좋다.