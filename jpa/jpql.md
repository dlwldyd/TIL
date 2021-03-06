# JPQL
```java
@Entity
public class Member {

    @Id
    @GeneratedValue
    private Long id;
    private String username;
    private int age;

    @ManyToOne(fetch = FetchType.LAZY)
    @JoinColumn(name = "TEAM_ID")
    private Team team;

    @Enumerated(value = EnumType.STRING)
    private MemberType type;

    ...

}
```
```java
@Entity
public class Team {

    @Id
    @GeneratedValue
    private Long id;
    private String name;

    @OneToMany(mappedBy = "team")
    private List<Member> members = new ArrayList<>();

    ...

}
```
```java
public enum MemberType {
    ADMIN, USER
}
```
```java
@Entity
@Table(name = "ORDERS")
public class Order {

    @Id
    @GeneratedValue
    private Long id;
    private int orderAmount;

    @Embedded
    private Address address;

    @ManyToOne(fetch = FetchType.LAZY)
    @JoinColumn(name = "PRODUCT_ID")
    private Product product;

    ...

}
```
```java
@Embeddable
public class Address {

    private String city;
    private String street;
    private String zipcode;

    ...

}
```
```java
@Entity
public class Product {

    @Id
    @GeneratedValue
    private Long id;
    private String name;
    private int price;
    private int stockAmount;

    ...

}
```
## getResultList(), getSingleList
### getResultList()
```java
List<Member> resultList = em.createQuery("select m from Member m", Member.class)
    .getResultList();
```
* 리스트를 받아온다. 만약 결과가 없으면 빈 리스트가 반환된다.
* 이렇게 받아온 모든 엔티티는 영속성 컨텍스트에서 관리된다.(엔티티를 받아왔을 때만)
### getSingleResult
```java
Member singleResult = em.createQuery("select m from Member m", Member.class)
    .getSingleResult();
```
* 값을 하나만 받아온다. 결과가 없으면 NoResultException, 결과가 하나가 아니면 NonUniqueResultException을 반환한다.
* getSingleResult()가 아닌 Spring Data JPA 의 추상화된 함수를 사용하면 결과가 없을 때 비어있는 Optional 로 반환해준다.
## TypeQuery, Query
### TypeQuery
```java
TypedQuery<Member> query = em.createQuery("select m from Member m", Member.class);
List<Member> resultList = query.getResultList();
```
* 반환 타입이 명확할 때 사용한다.
* new 명령어를 쓰지 않으면 반환 타입으로는 엔티티 타입만 사용 가능하다.
### Query
```java
Query query = em.createQuery("select m.username, m.age from Member m");
List resultList = query.getResultList();
Object singleResult = query.getSingleResult();
```
* 반환 타입이 명확하지 않을 때 사용한다.
* 반환타입은 Object 타입이나 Object 타입의 리스트가 반환된다.
## 파라미터 바인딩
```java
TypedQuery<Member> query1 = em.createQuery("select m from Member m where m.username = :username", Member.class);
query.setParameter("username", "member1");

List<String> nameList = Arrays.asList("member1", "member2");
TypedQuery<Member> query2 = em.createQuery("select m from Member m where m.username in :username", Member.class);
query.setParameter("username", nameList);
```
* ':변수명'을 사용하면 파라미터를 바인딩 할 수 있다.
* setParameter(변수명, 바인딩할 객체)를 사용하면 ':변수명'을 원하는 객체로 바인딩 할 수 있다.(여기서는 :username을 "member1"이라는 String 객체로 바인딩했다.)
* 컬렉션 객체를 파라미터 바인딩 할 수도 있다.
## 스칼라 타입 프로젝션
```java
public class MemberDTO {
    private String username;
    private int age;

    public MemberDTO(String username, int age) {
        this.username = username;
        this.age = age;
    }

    ...

}
```
```java
List resultList1 = em.createQuery("select m.username, m.age from Member m")
    .getResultList();
List<MemberDTO> resultList2 = em.createQuery("select new jpql.MemberDTO(m.username, m.age) from Member m", MemberDTO.class)
    .getResultList();
```
* 스칼라 타입을 받아올 때는 Query 써도 되지만 그렇게 하면 Object[]를 받아온다.(List\<Object[]>) 그래서 타입캐스팅을 해야하고 여러모로 불편한 부분이 많다. 따라서 Query 를 쓰는 것 보단 DTO 를 만들어 해당 객체에 값을 넣는게 좋다.
* jpql 에서는 엔티티에만 값을 넣을 수 있는데 엔티티가 아닌 객체에 값을 넣으려면 select 문에 new 명령어를 써야한다. 이때 DTO 가 들어있는 패키지 경로까지 모두 써야하고 값을 세팅해 주는 생성자도 필요하다.
* 패키지 경로까지 모두 써줘야하는 불편함은 QueryDSL을 사용하면 해소할 수 있다.
## 페이징 API
```java
List<Member> resultList = em.createQuery("select m from Member m order by m.age desc", Member.class)
    .setFirstResult(1)
    .setMaxResults(10)
    .getResultList();
```
* JPA가 페이징을 각 데이터베이스 방언에 맞추어 변형해 준다.
* setFirstResult : offset
* setMaxResults : row 개수
## 조인
### 내부 조인
```java
List<Member> resultList = em.createQuery("select m from Member m join m.team t on m.username = t.name", Member.class)
    .getResultList();
```
* 연관이 없는 값끼리 비교할 때 on 절을 사용하면 비교할 수 있다.
* 외래키와 기본키의 비교는 SQL로 변환될 때 자동으로 들어간다.
### 외부 조인
```java
List<Member> resultList = em.createQuery("select m from Member m left join m.team t on t.name = 'teamA'", Member.class)
    .getResultList();
```
### 세타 조인
```java
//Member와 Team은 연관관계가 있지만, 연관관계가 없더라도 사용 가능
List<Member> resultList = em.createQuery("select count(m) from Member m, Team t where m.username = t.name", Member.class)
    .getResultList();
```
* 연관관계가 없는 엔티티끼리 비교할 때 사용한다.
### 연관관계가 없는 내부 조인
```java
//Member와 Team은 연관관계가 있지만, 연관관계가 없더라도 사용 가능
//m.team t -> Team t
List<Member> resultList = em.createQuery("select m from Member m join Team t on m.username = t.name", Member.class)
    .getResultList();
```
* 하이버네이트 5.1버전 이상일 때 사용가능하다.
* 세타조인처럼 동작한다.
### 연관관계가 없는 외부 조인
```java
//Member와 Team은 연관관계가 있지만, 연관관계가 없더라도 사용 가능
//m.team t -> Team t
List<Member> resultList = em.createQuery("select m from Member m left join Team t on m.username = t.name", Member.class)
    .getResultList();
```
* 하이버네이트 5.1버전 이상일 때 사용가능하다.
* 연관관계가 없는 엔티티 끼리의 외부조인을 할 때 사용한다.
## 서브 쿼리 지원 함수
### (not) exists
```java
List<Member> resultList = em.createQuery("select m from Member m where exists (select t from m.team t where t.name = 'teamA')", Member.class)
    .getResultList();
```
* 서브쿼리에 결과가 존재하면 참
### all
```java
List<Order> resultList = em.createQuery("select o from Order o where o.orderAmount > all (select p.stockAmount from Product p)", Order.class)
    .getResultList();
```
* 모두 만족하면 참
### any, some
```java
List<Order> resultList = em.createQuery("select m from Member m where m.team = any (select t from Team t)", Order.class)
    .getResultList();
```
* 하나라도 만족하면 참
### (not) in
```java
List<Member> resultList = em.createQuery("select m from Member m where m.team in (select t from Team t)", Member.class)
    .getResultList();
```
* 서브쿼리 결과 중 하나라도 같은 것이 있으면 참
* JPQL에서 모든 서브 쿼리는 from 절에서 사용 불가능 하다.(JPA에서는 where, having 절에서만 사용 가능, 하이버네이트는 select 절에서도 사용 가능)
## 타입 표현
* 문자 : ex) 'hello'
* 숫자 : ex) 10L (Long), 10D (Double), 10F (Float)
* Boolean : ex) true, false
### enum
```java
List<Member> resultList = em.createQuery("select m.username, 'hello', true from Member m where m.type = jpql.MemberType.ADMIN", Member.class)
    .getResultList();
```
* enum 타입은 패키지 경로까지 적어줘야한다. 하지만 만약 파라미터 바인딩을 사용하면 패키지 경로를 안적어도 된다.
### 엔티티 타입(type, treat 사용)
```java
@Entity
@Inheritance(strategy = InheritanceType.JOINED)
@DiscriminatorColumn
public abstract class Item {

    ...

}
```
```java
@Entity
// @DiscriminatorValue("hello")
public class Book extends Item {

    ...

}
```
```java
List<Item> resultList1 = em.createQuery("select i from Item i where type(i) in (Book, Movie)", Item.class)
    .getResultList();

List<Item> resultList2 = em.createQuery("select i from Item i where type(i) = Album", Item.class)
    .getResultList();
List<Item> resultList3 = em.createQuery("select i from Item i where treat(i as Book).author", Item.class)
    .getResultList();
```
* 상속관계에서 사용한다.
* type(i) = Book 에서 Book은 DTYPE이 아니라 클래스 이름이 들어가야 한다. 만약 @DiscriminatorValue()를 통해 DTYPE에 들어갈 값을 바꿨으면 SQL로 변환 시 알아서 변환해준다.
* 객체에서 다운캐스팅처럼 사용할 때 treat 를 사용한다.(Item 객체를 Book 으로 다운캐스팅, author 는 Book 에만 있음)
## 조건식
### 기본 case식
```java
String queryString =
    "select " +
        "case when m.age <= 10 then '학생요금' " +
        "when m.age >=60 then '경로요금' " +
        "else '일반요금' " +
        "end " +
    "from Member m";
List<String> resultList = em.createQuery(queryString, String.class)
    .getResultList();
```
### 단순 case식
```java
String queryString =
    "select " +
        "case t.name " + 
        "when 'teamA' then '인센티브 110%' " +
        "when 'teamB' then '인센티브 120%' " +
        "else '인센티브 105%' " +
        "end " +
    "from Team t";
List<String> resultList = em.createQuery(queryString, String.class)
    .getResultList();
```
### coalesce
```java
List<String> resultList = em.createQuery("select coalesce(m.username, '이름 없는 회원') from Member m", String.class)
    .getResultList();
```
* 값이 있으면 값을 가져오고 null 이면 2번째 파라미터로 넣은 값이 반환된다.
### nullif
```java
List<String> resultList = em.createQuery("select nullif(m.username, '관리자') from Member m", String.class)
    .getResultList();
```
* m.username 이 관리자면 해당 값 대신 null 이 반환된다.
## 함수
### JPQL 기본 함수
* concat : select concat(m.username, m.username) from Member m -> 문자열 합침
* substring : select substring(m.username, 2, 3) from Member m -> 문자열 잘라서 반환
* locate : select locate('ber', m.username) from Member m -> 인덱스 반환
* lower, upper : select upper(m.username) from Member m -> 대소문자로 변환해서 반환
* abs, sqrt, mod : select abs(m.age) from Member m -> 절대값, 제곱근, 나머지 반환
* size : select size(t.members) from Team t -> 컬렉션의 크기를 반환
### 사용자 정의 함수
```java
public class MyH2Dialect extends H2Dialect {

    public MyH2Dialect() {
        registerFunction("group_concat", new StandardSQLFunction("group_concat", StandardBasicTypes.STRING));
    }
}
```
```xml
<property name="hibernate.dialect" value="dialect.MyH2Dialect"/>
```
```java
List<String> resultList = em.createQuery("select function('group_concat', m.username) from Member m", String.class)
    .getResultList();
```
* 내가 사용하는 데이터베이스의 dialect 를 상속하는 클래스를 만든 후 persistence.xml 이나 application.properties에 기존의 dialect 대신 내가 만든 클래스를 dialect 로 등록하면 사용자 정의 함수를 사용할 수 있다.
* function(함수명 , 함수에 들어갈 파라미터, 함수에 들어갈 파라미터, ... )를 통해 해당 사용자 정의 함수를 쓸 수 있다.
* 하이버네이트를 사용한다면 select group_concat(m.username) from Member m 처럼 function을 통해 함수를 쓰지 않아도 된다.
## 경로 표현식
### 상태 필드
```java
List<String> resultList = em.createQuery("select m.username from Member m", String.class)
    .getResultList();
```
* 경로 탐색의 끝, 더 이상 탐색이 불가능하다.
### 단일 값 연관 필드(@ManyToOne, @OneToOne)
```java
//묵시적 내부 조인 발생
List<Team> resultList1 = em.createQuery("select m.team from Member m", Team.class)
    .getResultList();

//명시적 조인
List<Team> resultList2 = em.createQuery("select m.team from Member m join m.team t", Team.class)
    .getResultList();
```
* 묵시적 내부조인 발생, 계속해서 탐색 가능하다.(m.team -> m.team.name)
* 묵시적 내부조인이 발생하면 SQL을 튜닝 하는데 가독성이 좋지 않으므로 명시적 조인을 하는 편이 더 좋다.
### 컬렉션 값 연관 필드(@OneToMany, @ManyToMany)
```java
//members에서 더 이상 경로탐색 불가
List<Collection> resultList1 = em.createQuery("select t.members from Team t", Collection.class)
    .getResultList();

//members의 별칭 m을 얻어서 경로탐색 가능, 명시적 조인
List<String> resultList2 = em.createQuery("select m.username from Team t join t.members m", String.class)
    .getResultList();
```
*  묵시적 내부조인 발생, 탐색이 더 이상 불가(t.members.size 를 통해 리스트 사이즈는 받을 수 있음)
* 묵시적 내부조인이 발생하면 SQL을 튜닝 하는데 가독성이 좋지 않으므로 명시적 조인을 하는 편이 더 좋다.
* from 절에서 명시적 조인을 통해 별칭을 얻으면 계속해서 탐색 가능하다.
## 페치 조인
```java
List<Member> resultList1 = em.createQuery("select m from Member m join fetch m.team", Member.class)
    .getResultList();

//이런 경우에는 페치 조인 대상에 별칭을 사용할 수도 있다.
List<Team> resultList2 = em.createQuery("select t from Team t join fetch t.members m join fetch m.team", Team.class)
    .getResultList();
```
* 즉시 로딩 전략 처럼 작동한다.(team을 조인해서 함께 조회)
* JPA 표준으로는 페치 조인 대상에는 별칭을 줄 수 없지만 하이버네이트를 사용하면 가능하다. 하지만 가급적이면 사용하지 않는 것이 좋다.
## 컬렉션 페치 조인
```java
List<Team> resultList11 = em.createQuery("select t from Team t join fetch t.members where t.name = 'teamA'", Team.class)
    .getResultList();

//중복 제거
List<Team> resultList = em.createQuery("select distinct t from Team t join fetch t.members where t.name = 'teamA'", Team.class)
    .getResultList();
```
<center><img src="./../img/collection_fetch_join.png"></center>

* resultList에 같은 엔티티가 존재할 수 있다.(같은 엔티티가 resultList에 있더라도 영속성 컨텍스트에선 하나로 관리)
* SQL 에서는 distinct 사용 시 같은 row 일 경우에만 중복이 제거되지만 JPQL 에서는 resultList에서 같은 엔티티일 때의 중복도 제거해준다.
* 둘 이상의 컬렉션은 페치 조인 할 수 없다.
* 컬렉션을 페치 조인 하면 페이징 API(setFirstResult(), setMaxResult())를 사용할 수 없다.
## BatchSize
```java
@Entity
public class Team {

    @Id
    @GeneratedValue
    private Long id;
    private String name;

    @BatchSize(size = 100)
    @OneToMany(mappedBy = "team")
    private List<Member> members = new ArrayList<>();

    ...

}
```
* 지연로딩 전략에서 select t from Team t 이라는 쿼리를 날리면 나중에 team.getMembers().getName() 등을 할 때 Member 테이블 에 대한 쿼리가 날라간다. 이 때 N명의 Member에 대한 team.getMembers().getName() 를 하면 N개의 쿼리가 날라간다. 하지만 @BatchSize 를 사용하면 team.getMembers().getName()을 할 때 원하는 Member 뿐만 아니라 @BatchSize 에서 정한 size 만큼 Member 테이블로 부터 데이터를 미리 받아온다.(select * from Member m where m.team_id  in (?, ?, ...) 에서 in 쿼리의 개수(?의 개수)가 @BatchSize 에서 정한 size 만큼 들어간다)
* persistence.xml 에 \<property name="hibernate.default_batch_fetch_size" value="100" /> 을 추가하거나 application.properties 에 hibernate.default_batch_fetch_size=100 을 추가하면 글로벌 설정으로 설정할 수 있다. 실무에서는 보통 글로벌 설정으로 가져간다.
## NamedQuery
### 어노테이션
```java
@Entity
@NamedQueries(
        {@NamedQuery(
                name = "Member.findByUsername",
                query = "select m from Member m where m.username = :username"
        ),
        @NamedQuery(
                name = "Member.findTeamByUsername",
                query = "select m.team from Member m join m.team where m.username = :username"
        )}
)
public class Member {

    ...

}
```
### xml
```xml
<!--META-INF/persistence.xml-->
<persistence-unit name="jpabook" >
    <mapping-file>META-INF/ormMember.xml</mapping-file>
```
```xml
<?xml version="1.0" encoding="UTF-8"?>
<entity-mappings xmlns="http://xmlns.jcp.org/xml/ns/persistence/orm" version="2.1">
    <named-query name="Member.findByUsername">
        <query><![CDATA[
            select m
            from Member m
            where m.username = :username]]>
        </query>
    </named-query>
    <named-query name="Member.count">
        <query>select count(m) from Member m</query>
    </named-query>
</entity-mappings>
```
```java
List<Member> resultList = em.createNamedQuery("Member.findByUserName", Member.class)
    .setParameter("username", "member1")
    .getResultList();
```
* 어플리케이션 로딩 시점에 쿼리를 검증하기 때문에 JPQL 문법오류를 컴파일 시점에 알 수 있다.
* 어플리케이션 로딩 시점에 쿼리를 파싱해서 SQL 로 변환해두기 때문에 속도가 빠르다.
## 벌크 연산
```java
int resultCount1 = em.createQuery("update m set m.age = 20")
    .executeUpdate();

int resultCount2 = em.createQuery("delete m from Member m where m.username = 'member1'")
    .executeUpdate();
```
* JPQL로 작성한 update, delete 쿼리를 말한다. 영속성 컨텍스트의 변경 감지 기능으로 실행하기에 너무 많은 update, delete 쿼리 실행 시 사용한다.
* 영향을 받은 엔티티의 개수를 반환받는다. 
* 벌크 연산은 영속성 컨텍스트를 무시하고 데이터베이스에 직접 쿼리를 보낸다.(영속성 컨텍스트에 반영X)
* 벌크 연산 후 영속성 컨텍스트를 초기화 시켜야한다.(벌크 연산 후 clear())
* 벌크 연산을 수행하면 수행하기 전에 flush 가 자동으로 수행되기 때문에 flush 는 고려하지 않아도 된다.