# 벌크성 쿼리, SQL function
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
## 벌크성 쿼리
### update
```java
@Test
void bulkUpdate() {

    //update Member m set m.username = '비회원' where m.age < 28
    long count = queryFactory
            .update(member)
            .set(member.username, "비회원")
            .where(member.age.lt(28))
            .execute();

    em.clear();

    List<Member> result = queryFactory
            .selectFrom(member)
            .fetch();
}

@Test
void bulkAdd() {

    //update Member m set m.age = m.age + 1
    long count = queryFactory
            .update(member)
            .set(member.age, member.age.add(1))
            .execute();
    em.clear();

    List<Member> result = queryFactory
            .selectFrom(member)
            .fetch();

    for (Member member : result) {
        System.out.println("member = " + member);
    }
}
```
* fetch()가 아닌 execute()를 실행해야 한다.
* JPQL과 마찬가지로 영속성 컨텍스트를 무시하고 데이터베이스에 쿼리가 나가기 때문에 update쿼리 후 반영된 결과를 원한다면 EntityManager를 clear()를 해줘야 한다.
### delete
```java
@Test
void bulkDelete() {

    //delete from Member m where m.age > 18
    long count = queryFactory
            .delete(member)
            .where(member.age.gt(18))
            .execute();

    em.clear();

    List<Member> result = queryFactory
            .selectFrom(member)
            .fetch();

    for (Member member : result) {
        System.out.println("member = " + member);
    }
}
```
* update와 마찬가지로 execute()를 실행시켜줘야하고 영속성 컨텍스트를 무시하고 쿼리가 나가기 때문에 쿼리가 반영된 결과를 원한다면 EntityManager를 clear()를 해줘야 한다.
## SQL function
```java
@Test
    void sqlFunction() {

        //select function('replace', m.username, 'member', 'M') from Member m
        List<String> result = queryFactory
                .select(Expressions.stringTemplate(
                        "function('replace', {0}, {1}, {2})",
                        member.username, "member", "M"))
                .from(member)
                .fetch();

        for (String s : result) {
            System.out.println("s = " + s);
        }
    }

    @Test
    void sqlFunction2() {

        //select m.username from Member m where m.username = lower(m.username)
        List<String> result = queryFactory
                .select(member.username)
                .from(member)
                .where(member.username.eq(
                       //Expressions.stringTemplate("function('lower', {0})", member.username)))
                        member.username.lower())) //ANSI 표준에 있는 문법은 어느정도 Querydsl 이 자체적으로 제공해줌
                .fetch();

        for (String s : result) {
            System.out.println("s = " + s);
        }
    }
```
* Expressions.stringTemplate("function(함수명, {0}, {1}, ...)", arg1, arg2, ...)을 통해 SQL function을 사용할 수 있다. 단, application.properties에 등록한 데이터베이스 dialect에 해당 함수가 등록되어 있어야 사용 가능하다.
* ANSI 표준에 있는 문법은 어느정도 Querydsl 이 자체적으로 제공해주기 때문에 Expressions.stringTemplate을 사용하지 않고도 사용 가능하다.