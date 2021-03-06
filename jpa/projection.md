# 프로젝션과 반환타입
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
## 프로젝션 대상이 하나일 때
```java
@Test
void simpleProjection() {
    List<String> result = queryFactory
            .select(member.username)
            .from(member)
            .fetch();

    for (String s : result) {
        System.out.println("s = " + s);
    }
}
```
* 프로젝션 대상이 하나이면 반환타입이 명확하므로 타입을 명확하게 지정할 수 있다.
## Tuple
```java
@Test
void tupleProjection() {
    List<Tuple> result = queryFactory
            .select(member.username, member.age)
            .from(member)
            .fetch();

    for (Tuple tuple : result) {
        String username = tuple.get(member.username);
        Integer age = tuple.get(member.age);
        System.out.println("username = " + username);
        System.out.println("age = " + age);
    }
}
```
* 프로젝션 대상이 둘 이상이면 DTO나 Tuple로 반환한다.
* Tuple 은 QueryDsl 종속적인 클래스이기 때문에 Repository 계층에서만 쓰고 Service 계층이나 Controller 계층에는 DTO 로 변환해서 넘겨주는 것이 좋다.(DB 관련 기술을 바꿀 일이 있을 때 Repository 계층으로만 변화를 한정할 수 있다.)
## DTO 반환
```java
@Getter
@Setter
@ToString
@NoArgsConstructor
public class MemberDto {

    private String username;
    private int age;

    //@QueryProjection 어노테이션을 생성자에 붙여주고 compileQuerydsl 을 실행하면 QMemberDto 가 생성된다.
    @QueryProjection
    public MemberDto(String username, int age) {
        this.username = username;
        this.age = age;
    }
}
```
```java
@Getter
@Setter
@ToString
@AllArgsConstructor
@NoArgsConstructor
public class UserDto {

    private String name;
    private int age;
}
```
```java
@Test
void findDtoByQuerydslWithSetter() {

    //select new orm.querydsl.dto.MemberDto(m.username, m.age) from Member m
    //new MemberDto()로 생성한 후 setter 로 필드값을 초기화(NoArgsConstructor, setter 필요)
    List<MemberDto> result = queryFactory
            .select(Projections.bean(MemberDto.class, //bean 은 자바빈(getter, setter)을 뜻함
                    member.username,
                    member.age))
            .from(member)
            .fetch();

    for (MemberDto memberDto : result) {
        System.out.println("memberDto = " + memberDto);
    }
}

@Test
void findDtoByQuerydslWithFields() {

    //select new orm.querydsl.dto.MemberDto(m.username, m.age) from Member m
    //필드값에 직접 값을 주입(필드가 private 라도 reflection 을 통해 직접 주입 가능)
    List<MemberDto> result = queryFactory
            .select(Projections.fields(MemberDto.class,
                    member.username,
                    member.age))
            .from(member)
            .fetch();

    for (MemberDto memberDto : result) {
        System.out.println("memberDto = " + memberDto);
    }
}

@Test
void findDtoByQuerydslWithConstructor() {

    //select new orm.querydsl.dto.MemberDto(m.username, m.age) from Member m
    //new MemberDto(m.username, m.age)로 객체 생성하면서 초기화
    List<MemberDto> result = queryFactory
            .select(Projections.constructor(MemberDto.class,
                    member.username,
                    member.age))
            .from(member)
            .fetch();

    for (MemberDto memberDto : result) {
        System.out.println("memberDto = " + memberDto);
    }
}

@Test
void findUserDtoByQuerydsl() {

    QMember member2 = new QMember("member2");
    /*
    필드로 주입해 초기화할 때는 필드명을 들어간 인자의 이름으로 판단해 주입한다. UserDto 에는 필드이름이 username 이
    아니라 name 인데 인자로는 member.username 이 들어갔기 때문에 만약 as()를 통해 alias 를 변경하지 않는다면
    UserDto 의 name 에는 null 값이 들어간다.
        */
    List<UserDto> result1 = queryFactory
            .select(Projections.fields(UserDto.class,
                    member.username.as("name"),
                    member.age))
            .from(member)
            .fetch();

    //setter 를 통해 초기화할 때도 setter 이름과 인자의 이름으로 판단하기 때문에 as()로 alias 를 변경해야한다.
    //서브쿼리에 alias 를 줘야할 때는 ExpressionUtils.as()를 통해 줄 수 있다.
    List<UserDto> result2 = queryFactory
            .select(Projections.bean(UserDto.class,
                    member.username.as("name"),
                    ExpressionUtils.as(JPAExpressions
                            .select(member2.age.max())
                            .from(member2), "age")
            ))
            .from(member)
            .fetch();

    for (UserDto userDto : result1) {
        System.out.println("userDto = " + userDto);
    }
    System.out.println(" =================================================== ");
    for (UserDto userDto : result2) {
        System.out.println("userDto = " + userDto);
    }
}

@Test
void findDtoByQueryProjection() {

    //constructor 와 달리 컴파일 시점에 오류를 잡을 수 있다.
    //DTO 가 Querydsl 라이브러리에 의존적이게 된다.
    List<MemberDto> result = queryFactory
            .select(new QMemberDto(member.username, member.age))
            .from(member)
            .fetch();

    for (MemberDto memberDto : result) {
        System.out.println("memberDto = " + memberDto);
    }
}
```
* Projections를 통해 DTO 반환이 가능하다.
* 자바빈 프로퍼티 방식 : setter 필요, NoArgsConstructor 필요
* 필드 주입 방식 : 필드가 private라도 reflection 기술을 이용해 필드에 직접적인 주입 가능
* 생성자 방식 : 생성자 정의 필요, 생성자의 파라미터 순서와 constructor()의 Args 순서가 같아야함
* 필드 주입 방식이나 생성자 방식을 사용할 때는 들어간 인자의 이름으로 어느 필드에 들어갈지 판단한다. 만약 들어간 인자의 alias가 필드 이름과 일치하지 않는다면 as()를 사용하여 alias의 이름을 필드 이름과 같게 맞춰주면 된다. 서브쿼리의 경우에는 ExpressionUtils.as(서브쿼리, alias)를 사용해 alias를 필드 이름과 같게 맞춰주면 된다.
* @QueryProjection 어노테이션을 생성자에 적용하고 compileQuerydsl 을 실행하면 QMemberDto가 생성되는데 이것을 통해 DTO반환을 간단하게 할 수 있다. 단, 이 방법을 사용하면 컴파일 시점에 오류를 잡을 수는 있지만 DTO가 QueryDSL 라이브러리에 의존적이게 된다.