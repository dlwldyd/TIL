# 동적 쿼리
```java
@Entity
@Getter
@Setter
@NoArgsConstructor(access = AccessLevel.PROTECTED)
@ToString
public class Member {

    @Id
    @GeneratedValue
    @Column(name = "member_id")
    private Long id;

    private String username;

    private int age;

    @ToString.Exclude
    @ManyToOne(fetch = FetchType.LAZY)
    @JoinColumn(name = "team_id")
    private Team team;

    public Member(String username) {
        this(username, 0);
    }

    public Member(String username, int age) {
        this(username, age, null);
    }

    public Member(String username, int age, Team team) {
        this.username = username;
        this.age = age;
        if (team != null) {
            changeTeam(team);
        }
    }

    public void changeTeam(Team team) {
        this.team = team;
        team.getMembers().add(this);
    }
}
```
```java
@Entity
@Getter
@Setter
@NoArgsConstructor(access = AccessLevel.PROTECTED)
@ToString
public class Team {

    @Id
    @GeneratedValue
    @Column(name = "team_id")
    private Long id;

    private String name;

    @ToString.Exclude
    @OneToMany(mappedBy = "team")
    private List<Member> members = new ArrayList<>();

    public Team(String name) {
        this.name = name;
    }
}
```
```java
@BeforeEach
void before() {
    queryFactory = new JPAQueryFactory(em);
    Team teamA = new Team("teamA");
    Team teamB = new Team("teamB");
    em.persist(teamA);
    em.persist(teamB);

    Member member1 = new Member("member1", 10, teamA);
    Member member2 = new Member("member2", 20, teamA);
    Member member3 = new Member("member3", 30, teamB);
    Member member4 = new Member("member4", 40, teamB);
    em.persist(member1);
    em.persist(member2);
    em.persist(member3);
    em.persist(member4);
}
```
## BooleanBuilder
```java
@Test
void dynamicQuery_booleanBuilder() {
    String usernameParam = "member1";
    Integer ageParam = 10;

    List<Member> result = searchMember1(usernameParam, ageParam);
    assertThat(result.size()).isEqualTo(1);
}

private List<Member> searchMember1(String usernameCond, Integer ageCond) {

    //new BooleanBuilder(member.username.eq(usernameCond)) 처럼 초기조건을 넣을 수도 있음
    BooleanBuilder builder = new BooleanBuilder();
    if (usernameCond != null) {
        //빌더에 .and(member.username.eq(usernameCond))가 들어감
        //빌더에 초기 조건이 아무것도 없을 때 들어가면 member.username.eq(usernameCond)가 들어감
        builder.and(member.username.eq(usernameCond));
    }

    if (ageCond != null) {
        builder.and(member.age.eq(ageCond));
    }

    return queryFactory
            .selectFrom(member)
            .where(builder) //builder.or(member.username.eq("member10"))처럼 builder 뒤에도 조건을 덧붙일 수 있음
            .fetch();
}
```
* where절에 아무 조건도 붙지 않은 BooleanBuilder가 들어가면 where절은 무시된다.(JPQL에 where 절이 붙지 않음)
* BooleanBuilder에 builder.or()처럼 추가적인 조건을 계속해서 덧붙일 수 있다.
* new BooleanBuilder(member.age.gt(20))처럼 초기조건을 생성시점에 부여할 수 있다.
## where절을 이용한 동적 쿼리
```java
@Test
void dynamicQuery_whereParam() {
    String usernameParam = "member1";
    Integer ageParam = 10;

    List<Member> result = searchMember2(usernameParam, ageParam);
//        List<Member> result = searchMember2(null, null);
    assertThat(result.size()).isEqualTo(1);
}

private List<Member> searchMember2(String usernameCond, Integer ageCond) {

    return queryFactory
            .selectFrom(member)
//                .where(usernameEq(usernameCond), ageEq(ageCond))
            .where(allEq(usernameCond, ageCond))
            .fetch();
}

//BooleanBuilder 처럼 조립하기 위해 BooleanExpression 으로 반환하는 것이 좋다.
private BooleanExpression usernameEq(String usernameCond) {
    if (usernameCond == null) {
        return null;
    }
    return member.username.eq(usernameCond);
}

private BooleanExpression ageEq(Integer ageCond) {
    if (ageCond == null) {
        return null;
    }
    return member.age.eq(ageCond);
}

private BooleanExpression allEq(String usernameCond, Integer ageCond) {
    if (usernameCond == null && ageCond == null) {
        return null;
    }
    return usernameEq(usernameCond).and(ageEq(ageCond));
}
```
* where절에 파라미터로 null이 들어가면 조건이 무시되는 것을 이용한 동적 쿼리 방식이다. where절에 들어갈 파라미터를 반환하는 메서드(null 혹은 조건(BooleanExpression)을 반환)를 만들어 동적 쿼리를 만든다.
* Predicate를 반환해도 되지만 BooleanBuilder 에서 builder.or()처럼 계속해서 조건을 덧붙일 수 있게 하려면 BooleanExpression을 반환하는 것이 좋다.
* 이렇게 메서드로 만든 조건을 재사용할 수 있기 때문에 재사용성에서도 장점이 있다.