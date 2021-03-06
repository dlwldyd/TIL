# QueryDSL 사용해서 Page<>반환
```java
public class MemberRepositoryCustomImpl implements MemberRepositoryCustom{

    @Override
    public Page<MemberTeamDto> searchPage(MemberSearchCondition condition, Pageable pageable) {
        List<MemberTeamDto> result = queryFactory
                .select(new QMemberTeamDto(
                        member.id,
                        member.username,
                        member.age,
                        team.id,
                        team.name))
                .from(member)
                .join(member.team, team)
                .where(
                        usernameEq(condition.getUsername()),
                        teamNameEq(condition.getTeamName()),
                        ageGoe(condition.getAgeGoe()),
                        ageLoe(condition.getAgeLoe()))
                .offset(pageable.getOffset())
                .limit(pageable.getPageSize())
                .fetch();

        JPAQuery<Long> countQuery = queryFactory
                .select(member.count())
                .from(member)
                .leftJoin(member.team, team)
                .where(
                        usernameEq(condition.getUsername()),
                        teamNameEq(condition.getTeamName()),
                        ageGoe(condition.getAgeGoe()),
                        ageLoe(condition.getAgeLoe()));

        return PageableExecutionUtils.getPage(result, pageable, () -> countQuery.fetchOne());
    }

    private BooleanExpression usernameEq(String username) {
        return StringUtils.hasText(username) ? member.username.eq(username) : null;
    }

    private BooleanExpression teamNameEq(String teamName) {
        return StringUtils.hasText(teamName) ? team.name.eq(teamName) : null;
    }

    private BooleanExpression ageGoe(Integer ageGoe) {
        return ageGoe != null ? member.age.goe(ageGoe) : null;
    }

    private BooleanExpression ageLoe(Integer ageLoe) {
        return ageLoe != null ? member.age.loe(ageLoe) : null;
    }
}
```
* 스프링 데이터의 Page객체를 반환하고 싶을 때는 QueryDSL의 offset()과 limit을 이용한 페이징 쿼리와 count()를 이용한 카운트 쿼리를 따로 날려야한다.
* 반환할 Page<>객체는 new PageImpl<>(content 객체, Pageable 객체, 카운트 쿼리 결과)보다는 PageableExecutionUtils.getPage(content 객체, Pageable 객체, 카운트쿼리(람다식))를 통해 만들어 준다.
* PageableExecutionUtils.getPage()의 장점
  + 페이지 시작이면서 컨텐츠 사이즈가 페이지 사이즈보다 작을 때 카운트 쿼리를 날리지 않는다.(컨텐츠 사이즈가 카운트 쿼리의 결과와 같기 때문에 카운트 쿼리를 날릴 필요가 없기 때문)
  + 마지막 페이지 일 때 카운트 쿼리를 날리지 않는다.(현재 페이지 * 페이지 사이즈 + 컨텐츠 사이즈가 카운트 쿼리의 결과와 같기 때문에 카운트 쿼리를 날릴 필요가 없기 때문)
* Slice<>의 경우에는 카운트 쿼리를 날리지 않기 때문에 SliceImpl<>()을 사용하면 된다.