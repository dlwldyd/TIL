# 페이징
```java
@GetMapping("/members")
public Page<MemberDto> list(@PageableDefault(size = 10, sort = "username", direction = Sort.Direction.DESC) @Qualifier("member") Pageable memberPageable, @Qualifier("team") Pageable teamPageable) {
    Page<Member> memberPage = memberRepository.findAll(memberPageable);
    Page<Team> teamPage = teamRepository.findAll(teamPageable);

    Page<MemberDto> map = page.map(member ->
            new MemberDto(member.getId(), member.getUsername(), null));
    return map;
}
```
* Page<>를 반환하여 페이지 정보를 반환할 수 있다.
* @PageableDefault 어노테이션을 사용하여 해당 핸들러메서드에 대해서만 기본 페이징 설정을 바꿀 수 있다.
* @Qualifier 어노테이션을 사용하여 url 파라미터를 통해 여러 페이징 정보를 받을 수 있다.(@Qualifier에는 접두사명을 넣는다)
* 핸들러 메서드의 파라미터로 Pageable을 받으면 스프링이 url 파라미터로 받은 페이징 정보를 알아서 바인딩 해준다.
* Page<>를 JSON으로 response를 보내면 마지막에 페이지 정보가 붙어서 온다.
```
localhost:8080/members

localhost:8080/members?member_page=0
```
* default로 데이터는 20개씩 받는다.(위의 두 url은 같은 결과를 가진다)
* application.properties에 spring.data.web.pageable.default-age-size을 통해 글로벌로 기본 페이지 사이즈 설정이 가능하다.
```
localhost:8080/members?member_page=5

localhost:8080/members?member_page=2&member_size=3

localhost:8080/members?member_sort=username,desc&member_sort=age,desc
```
* page 지정해서 원하는 페이지 받을 수 있다.
* page의 시작은 0부터 시작한다.
* size로 데이터의 개수를 지정 가능하다.
* sort로 원하는 정렬 조건을 넣을 수 있다. 다만 정렬 조건이 복잡해지면 내부적으로 정렬을 구현해야한다.
* @Qualifier을 통해 여러 엔티티의 페이징을 할 때는 @Qualifier에 넣었던 접두사를 통해 페이징을 처리한다.(@Qualifier 사용X : `localhost:8080?page=1` -> @Qualifier("member") 사용 시 : `localhost:8080?member_page=1`)