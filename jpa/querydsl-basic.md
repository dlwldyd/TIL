# QueryDSL 기본
## QueryDSL 설정
```gradle
//맨 위에 넣어야함
buildscript {
	ext {
		queryDslVersion = "5.0.0" //스프링 부트 레퍼런스에 querydsl-apt, querydsl-jpa 알맞은 버전이 뭔지 나옴
	}
}

plugins {
	id 'org.springframework.boot' version '2.6.3'
	id 'io.spring.dependency-management' version '1.0.11.RELEASE'
	//querydsl 추가
	id "com.ewerk.gradle.plugins.querydsl" version "1.0.10"
	id 'java'
}

group = 'orm'
version = '0.0.1-SNAPSHOT'
sourceCompatibility = '11'

configurations {
	compileOnly {
		extendsFrom annotationProcessor
	}
}

repositories {
	mavenCentral()
}

dependencies {
	implementation 'org.springframework.boot:spring-boot-starter-data-jpa'
	implementation 'org.springframework.boot:spring-boot-starter-web'
	implementation 'com.github.gavlyukovskiy:p6spy-spring-boot-starter:1.5.8'

	//querydsl 라이브러리 추가
	implementation "com.querydsl:querydsl-jpa:${queryDslVersion}"
	implementation "com.querydsl:querydsl-apt:${queryDslVersion}"

	compileOnly 'org.projectlombok:lombok'
	runtimeOnly 'com.h2database:h2'
	annotationProcessor 'org.projectlombok:lombok'
	testImplementation 'org.springframework.boot:spring-boot-starter-test'
}

tasks.named('test') {
	useJUnitPlatform()
}

//querydsl 추가 시작

/*
Q파일이 만들어질 경로, Q파일은 엔티티가 수정되면 새로 생성해야 하기 때문에 git으로 관리하면 안된다.
그렇기 때문에 일반적으로 gitignore에 등록되어 있는 build 디렉토리에 Q파일을 저장한다.
 */
def querydslDir = "$buildDir/generated/querydsl"

querydsl {
	jpa = true
	querydslSourcesDir = querydslDir
}
sourceSets {
	main.java.srcDir querydslDir
}
compileQuerydsl{
	options.annotationProcessorPath = configurations.querydsl
}
configurations {
	compileOnly {
		extendsFrom annotationProcessor
	}
	querydsl.extendsFrom compileClasspath
}
//querydsl 추가 끝
```
* 설정 순서
  1. 위의 설정을 gradle.build에 추가해준다.
  2. Gradle -> Tasks -> build -> clean
  3. Gradle -> Tasks -> other -> compileQuerydsl
* 만약 처음 Q파일을 생성한다면 2번은 생략해도 된다.
## QueryDSL 기본 문법
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
### Q타입 생성
```java
@Test
public void startQuerydsl() {
//        QMember m = new QMember("m"); -> 같은 테이블을 조인하는 경우에는 써야할 수도 있음
//        QMember m = QMember.member;

    //select m from Member m where m.username = 'member1'
    Member findMember = queryFactory
            .selectFrom(member) // select(member).from(member) == selectFrom(member)
            .where(member.username.eq("member1"))
            .fetchOne();

    assertThat(findMember.getUsername()).isEqualTo("member1");
}
```
* new QMember()를 통해 Q타입을 생성할 수 있다. 이 때 생성자의 파라미터로 원하는 alias를 줄 수 있다. 같은 테이블을 조인하거나 할 때 사용한다.
* QMember.member를 통해 Q타입에 접근할 수 있다. 일반적으로 이 방법을 가장 많이 사용한다.
### 검색 조건 쿼리
```java
@Test
void search() {

    //select m from Member m where m.username = 'member1' and m.age = 10
    Member findMember = queryFactory
            .selectFrom(member)
            .where(member.username.eq("member1").and(member.age.eq(10)))
            // member.username.eq("member1") : username = 'member1'
            // member.username.ne("member1") :username != 'member1'
            // member.username.eq("member1").not() : username != 'member1'
            // member.username.isNotNull() : 이름이 is not null
            // member.age.in(10, 20) : age in (10,20)
            // member.age.notIn(10, 20) : age not in (10, 20)
            // member.age.between(10,30) : between 10, 30
            // member.age.goe(30) : age >= 30
            // member.age.gt(30) : age > 30
            // member.age.loe(30) : age <= 30
            // member.age.lt(30) : age < 30
            // member.username.like("member%") : like 검색
            // member.username.contains("member") : like ‘%member%’ 검색
            // member.username.startsWith("member") : like ‘member%’ 검색
            .fetchOne();
    assertThat(findMember.getUsername()).isEqualTo("member1");
    assertThat(findMember.getAge()).isEqualTo(10);
}
```
* JPQL이 제공하는 모든 검색 조건을 제공한다.
```java
@Test
void searchAndParam() {

    //select m from member m where m.username = 'member1' and m.age = 10
    Member findMember = queryFactory
            .selectFrom(member)
            .where(
                    member.username.eq("member1"),
                    member.age.eq(10)
            )
            .fetchOne();
    assertThat(findMember.getUsername()).isEqualTo("member1");
    assertThat(findMember.getAge()).isEqualTo(10);
}
```
* where()에 파라미터로 검색조건을 여러 개 추가하면 AND로 조건이 추가된다.
* 파라미터 값이 null일 경우 무시된다.
### 결과 조회
```java
@Test
void resultFetchTest() {

    //getResultList
    List<Member> fetch = queryFactory
            .selectFrom(member)
            .fetch();

    //getSingleResult
    Member fetchOne = queryFactory
            .selectFrom(member)
            .fetchOne();

    // .fetchFirst() == .limit(1).fetchOne()
    Member fetchFirst = queryFactory
            .selectFrom(member)
            .fetchFirst();
}
```
* fetch() : 리스트 조회, 데이터가 없으면 빈 리스트를 반환한다.
* fetchOne() : 단 건 조회, 결과가 없으면 null을 반환한다. 결과가 둘 이상이면 com.querydsl.core.NonUniqueResultException이 발생한다.
* fetchFirst() : limit(1).fetchOne()와 같다.
### 정렬
```java
@Test
void sort() {
    em.persist(new Member(null, 100));
    em.persist(new Member("member5", 100));
    em.persist(new Member("member6", 100));

    //select m from Member m where m.age = 100 order by m.age desc order by m.username asc nulls last
    List<Member> result = queryFactory
            .selectFrom(member)
            .where(member.age.eq(100))
            .orderBy(member.age.desc())
            .orderBy(member.username.asc().nullsLast())
            .fetch();

    Member member5 = result.get(0);
    Member member6 = result.get(1);
    Member memberNull = result.get(2);
    assertThat(member5.getUsername()).isEqualTo("member5");
    assertThat(member6.getUsername()).isEqualTo("member6");
    assertThat(memberNull.getUsername()).isNull();
}
```
* orderBy() 로 정렬 가능하다.
* desc() , asc() : 일반 정렬
* nullsLast() , nullsFirst() : null 데이터 순서 부여
### 페이징
```java
@Test
void paging() {

    //select m from Member m order by m.username desc, setFirstResult(0), setMaxResults(2)
    List<Member> result = queryFactory
            .selectFrom(member)
            .orderBy(member.username.desc())
            .offset(0) // 페이징은 0부터 시작
            .limit(2)
            .fetch();

    assertThat(result.size()).isEqualTo(2);
}
```
* offset() : 페이징 시작 지점, 0부터 시작
* limit() : row의 개수
### 집합 함수
```java
@Test
void aggregation() {

    //select count(m), sum(m.age), avg(m.age), max(m.age), min(m.age) from Member m
    Tuple tuple = queryFactory
            .select(member.count(),
                    member.age.sum(),
                    member.age.avg(),
                    member.age.max(),
                    member.age.min())
            .from(member)
            .fetchOne();

    assertThat(tuple.get(member.count())).isEqualTo(4);
    assertThat(tuple.get(member.age.sum())).isEqualTo(100);
    assertThat(tuple.get(member.age.avg())).isEqualTo(25);
    assertThat(tuple.get(member.age.max())).isEqualTo(40);
    assertThat(tuple.get(member.age.min())).isEqualTo(10);
}
```
* JPQL이 제공하는 모든 집합 함수를 제공한다.
### group by, having
```java
@Test
void groupBy() {

    //select t.name, avg(m.age) from Member m join m.team t group by t.name having t.name like 'team%'
    List<Tuple> result = queryFactory
            .select(team.name, member.age.avg())
            .from(member)
            .join(member.team, team)
            .groupBy(team.name)
            .having(team.name.startsWith("team"))
            .fetch();

    Tuple teamA = result.get(0);
    Tuple teamB = result.get(1);

    assertThat(teamA.get(team.name)).isEqualTo("teamA");
    assertThat(teamA.get(member.age.avg())).isEqualTo(15);

    assertThat(teamB.get(team.name)).isEqualTo("teamB");
    assertThat(teamB.get(member.age.avg())).isEqualTo(35);
}
```
* groupBy() : group by 절 사용
* having() : having 절 사용
### 일반 조인
```java
@Test
void join() {

    //select m from Member m join m.team t where t.name = 'teamA'
    List<Member> result = queryFactory
            .selectFrom(member)
            .join(member.team, team)
            .where(team.name.eq("teamA"))
            .fetch();


/* alias 를 바꾸고 싶을 때

    QTeam hello = new QTeam("hello");

    List<Member> result = queryFactory
            .selectFrom(member)
            .join(member.team, hello)
            .where(hello.name.eq("teamA"))
            .fetch();
*/

    assertThat(result)
            .extracting("username")
            .containsExactly("member1", "member2");
}
```
* join() : inner join 사용
* leftJoin() : left outer join 사용
* rightJoin() : right outer join 사용
### 세타 조인
```java
@Test
void theta_join() {
    em.persist(new Member("teamA"));
    em.persist(new Member("teamB"));
    em.persist(new Member("teamC"));

    //select m from Member m, Team t where m.username = t.name
    List<Member> result = queryFactory
            .select(member)
            .from(member, team)
            .where(member.username.eq(team.name))
            .fetch();

    assertThat(result)
            .extracting("username")
            .containsExactly("teamA", "teamB");
}
```
* where 절을 이용하여 세타조인 가능
* 연관관계가 없는 엔티티끼리 조인할 때 세타조인을 사용한다.
### on 절
```java
@Test
void join_on_filtering() {

    //select m, t from Member m left join m.team t on t.name = 'teamA'
    List<Tuple> result = queryFactory
            .select(member, team)
            .from(member)
            .leftJoin(member.team, team)
            .on(team.name.eq("teamA"))
            .fetch();
}
```
* inner join의 경우에는 on 절을 사용하여 필터링 하든 where 절을 사용하여 필터링 하든 같은 결과를 같기 때문에 굳이 on 절을 사용할 필요가 없다.
* outer join의 경우에는 on절로 필터링 하지 못한 row에는 null값이 들어간다.(팀 이름이 teamA가 아닌 멤버의 팀 컬럼에는 null값이 들어감)
### 연관관계가 없는 엔티티간 외부조인
```java
@Test
void join_on_no_relation() {
    em.persist(new Member("teamA"));
    em.persist(new Member("teamB"));
    em.persist(new Member("teamC"));

    //select m, t from Member m left join Team t on t.name = m.username
    List<Tuple> result = queryFactory
            .select(member, team)
            .from(member)
            .leftJoin(team)
            .on(team.name.eq(member.username))
            .fetch();
}
```
* leftJoin()에 Q타입만 파라미터로 넣어서 연관관계가 없는 엔티티간 외부조인을 할 수 있다.(leftJoin(member.team, team) -> leftJoin(team) : m.team t -> Team t)
### 페치 조인
```java
@Test
void fetchJoin() {
    em.flush();
    em.clear();

    // select m from Member m join fetch m.team t where m.username = 'member1'
    Member findMember = queryFactory
            .selectFrom(member)
            .join(member.team, team).fetchJoin()
            .where(member.username.eq("member1"))
            .fetchOne();

    assertThat(emf.getPersistenceUnitUtil().isLoaded(findMember.getTeam())).isTrue();
}
```
* fetchJoin()을 join()뒤에 붙여서 페치조인을 할 수 있다.
### 서브쿼리
```java
@Test
void subQuery() {

    QMember member2 = new QMember("member2");

    //select m from Member m where m.age = (select max(m2.age) from Member m2)
    List<Member> result = queryFactory
            .selectFrom(member)
            .where(member.age.eq(JPAExpressions.
                    select(member2.age.max())
                            .from(member2)
            ))
            .fetch();

    assertThat(result).extracting("age").containsExactly(40);
}

@Test
void subQuery2() {

    QMember member2 = new QMember("member2");

    //select m from Member m where m.age >= (select avg(m2.age) from Member m2)
    List<Member> result = queryFactory
            .selectFrom(member)
            .where(member.age.goe(JPAExpressions.
                    select(member2.age.avg())
                            .from(member2)
            ))
            .fetch();

    assertThat(result).extracting("age").containsExactly(30, 40);
}

@Test
void subQueryIn() {

    QMember member2 = new QMember("member2");

    //select m from Member m where m.age in (select m2.age from Member m2 where m2.age > 10)
    List<Member> result = queryFactory
            .selectFrom(member)
            .where(member.age.in(JPAExpressions.
                    select(member2.age)
                            .from(member2)
                            .where(member2.age.gt(10))
            ))
            .fetch();

    assertThat(result).extracting("age").containsExactly(20, 30, 40);
}

@Test
void selectSubQuery() {

    QMember member2 = new QMember("member2");

    //select m.username, (select avg(m2.age) from Member m2) from Member m
    List<Tuple> result = queryFactory
            .select(member.username,JPAExpressions.
                    select(member2.age.avg())
                            .from(member2))
            .from(member)
            .fetch();
}
```
* JPAExpressions를 이용해 서브쿼리를 사용할 수 있다.
* where 절에서 서브쿼리를 사용할 수 있다.
* 하이버네이트 사용시 select 절에서 서브쿼리를 사용할 수 있다.
* from 절에서는 서브쿼리를 사용할 수 없다.
### case 문
```java
//간단한 조건일 때
@Test
void simpleCase() {

    /*
    select
        case
            when m.age = 10 then '열살'
            when m.age = 20 then '스무살'
            else '기타'
        end
    from Member m
    */
    List<String> result = queryFactory
            .select(member.age
                    .when(10).then("열살")
                    .when(20).then("스무살")
                    .otherwise("기타"))
            .from(member)
            .fetch();
}

//복잡한 조건일 때
@Test
void complexCase() {

    /*
    select
        case
            when (m.age between 0 and 20) then '0~20살'
            when (m.age between 20 and 30) then '20~30살'
            else '기타'
        end
    from Member m
    */
    List<String> result = queryFactory
            .select(new CaseBuilder()
                    .when(member.age.between(0, 20)).then("0~20살")
                    .when(member.age.between(20, 30)).then("20~30살")
                    .otherwise("기타"))
            .from(member)
            .fetch();
}

@Test
void caseInOrderBy() {

    /*
    select 
        m.username, 
        m.age,
        case
            when (m.age between 0 and 20) then 2
            when (m.age between 21 and 30) then 1
            else 3
        end
    from
        Member m
    order by
        case
            when (m.age between 0 and 20) then 2
            when (m.age between 21 and 30) then 1
            else 3
        end desc 
    */
    NumberExpression<Integer> rankPath = new CaseBuilder()
            .when(member.age.between(0, 20)).then(2)
            .when(member.age.between(21, 30)).then(1)
            .otherwise(3);

    List<Tuple> result = queryFactory
            .select(member.username, member.age, rankPath)
            .from(member)
            .orderBy(rankPath.desc())
            .fetch();
}
```
* 간단한 조건일 때는 caseBuilder 없이 바로 쓸 수 있다.
* 복잡한 조건일 경우에는 casBuilder를 통해 case문을 만들어야 한다.
* caseBuilder를 통해 생성한 case문을 order by 절에서 사용할 수 있다.
### 상수 덧붙이기
```java
@Test
void constant() {

    //select m.username from Member m
    //상수는 쿼리결과 받은 후 붙여짐
    List<Tuple> result = queryFactory
            .select(member.username, Expressions.constant("A"))
            .from(member)
            .fetch();

    for (Tuple tuple : result) {
        System.out.println("tuple = " + tuple);
    }
}
```
* Expressions.contant()를 사용해 쿼리 결과에 상수를 덧붙일 수 있다.
### 문자 더하기
```java
@Test
void concat() {

    //select concat(concat(m.username, '_'), m.age) from Member m
    List<String> result = queryFactory
            .select(member.username.concat("_").concat(member.age.stringValue())) //{username}_{age}
            .from(member)
            .fetch();

    for (String s : result) {
        System.out.println("s = " + s);
    }
}
```
* concat 은 String 타입만 적용되기 때문에 StringValue() 로 String 타입으로 바꿔줘야한다. stringValue()는 enum 타입을 처리할 때 자주 사용한다.
* jpql 상으로는 int 타입을 String 타입으로 캐스팅 하지 않지만 sql 쿼리는 integer 타입을 char 타입으로 캐스팅 하는 쿼리를 보낸다.