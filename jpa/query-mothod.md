# 쿼리 메서드
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

    @ManyToOne(fetch = FetchType.LAZY)
    @JoinColumn(name = "team_id")
    @ToString.Exclude
    private Team team;

    @Version
    private int version;

    public Member(String username) {
        this.username = username;
    }

    public Member(String username, int age, Team team) {
        this.username = username;
        this.age = age;
        if (team != null) {
            changeTeam(team);
        }
    }

    public Member(String username, int age) {
        this.username = username;
        this.age = age;
    }

    public void changeTeam(Team team) {
        if (this.team != null) {
            this.team.getMembers().remove(this);
        }
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

    @OneToMany(mappedBy = "team", fetch = FetchType.LAZY)
    private List<Member> members = new ArrayList<>();

    public Team(String name) {
        this.name = name;
    }

    @Version
    private int version;
}
```
## 반환 타입
```java
public interface MemberRepository extends JpaRepository<Member, Long> {
    
    //컬렉션 반환
    List<Member> findListByUsername(String username);

    //단건 반환
    Member findMemberByUsername(String username);

    //Optional 반환
    Optional<Member> findOptionalByUsername(String username);
}
```
* 단건을 반환할 때는 결과가 하나인 것이 보장되어야 한다. 2개 이상이면 IncorrectResultSizeDataAccessException 예외터진다. JPA 에서는 NonUniqueResultException 이 터지지만(getSingleResult()) 스프링 데이터 JPA 가 스프링의 예외로 바꿔서 반환한다.
* JPA 에서는 결과가 없으면 NoResultException 예외가 터지지만 스프링 데이터 JPA 는 null 로 반환해준다.
* 단건 조회를 Optional 로 감싸서 반환할 수도 있다. 만약 반환 결과가 null일 수 있으면 Optional로 반환하는 것이 좋다.
## @Query
```java
public interface MemberRepository extends JpaRepository<Member, Long> {
    
    @Query("select m from Member m where m.username = :username and m.age = :age")
    List<Member> findMember(@Param("username") String username, @Param("age") int age);

    @Query("select m.username from Member m")
    List<String> findUsernameList();

    @Query("select new springdata.datajpa.dto.MemberDto(m.id, m.username, t.name) from Member m join m.team t")
    List<MemberDto> findMemberDto();

    @Query("select m from Member m where m.username in :names")
    List<Member> findByNames(@Param("names") Collection<String> names);
}
```
* @Query()에 있는 JPQL은 이름 없는 Named Query라 볼 수 있다.
* 파라미터를 바인딩 할 때는 @Param()을 사용해서 바인딩 할 수 있다.
## 페이징
```java
public interface MemberRepository extends JpaRepository<Member, Long> {

    @Query(value = "select m from Member m left join m.team t where m.age = :age",
            countQuery = "select count(m) from Member m where m.age = :age")
    Page<Member> findPageByAge(@Param("age") int age, Pageable pageable);

    Slice<Member> findSliceByAge(int age, Pageable pageable);
}
```
```java
@Test
public void paging() {

    memberRepository.save(new Member("member1", 10));
    memberRepository.save(new Member("member2", 10));
    memberRepository.save(new Member("member3", 10));
    memberRepository.save(new Member("member4", 10));
    memberRepository.save(new Member("member5", 10));

    int age = 10;

    //PageRequest 는 Pageable 의 구현체, 정렬 가능
    //(가져올 페이지, 페이지 크기(limit), 정렬과 관련) ->  파라미터를 이순으로 넣는다.
    //정렬 조건이 복잡해지면 그냥 @Query 로 직접 짜면 된다.
    PageRequest pageRequest = PageRequest.of(0, 3, Sort.by(Sort.Direction.DESC, "username"));

    Page<Member> page = memberRepository.findPageByAge(age, pageRequest);
    
    Slice<Member> slice = memberRepository.findSliceByAge(age, pageRequest);
    List<Member> listByAge = memberRepository.findListByAge(age, pageRequest);

    //map 메서드로 DTO 로 변환도 쉽게 가능하다.
    //스트림으로 바꿔서 하면 Page<MemberDto> 로 변환이 어려움
    //이거를 JSON 으로 반환하면 DTO 의 필드 말고도 totalPages, totalElement 등이 전부 JSON 으로 변환됨
    Page<MemberDto> map = page.map(m -> new MemberDto(m.getId(), m.getUsername(), null));

    //페이징해서 쿼리한 결과를 반환함
    List<Member> content = page.getContent();
    //전체 데이터의 row 수를 반환함, Slice 에는 없음
    long totalElements = page.getTotalElements();
    //현재 페이지의 번호를 반환함(0부터 시작)
    int number = page.getNumber();
    //전체 페이지 개수 반환함, Slice 에는 없음
    int totalPages = page.getTotalPages();
    //현재 페이지가 첫번째 페이지인지 반환함
    boolean first = page.isFirst();
    //다음 페이지가 존재하는지 반환함
    boolean hasNext = page.hasNext();

    assertThat(content.size()).isEqualTo(3);
    assertThat(totalElements).isEqualTo(5);
    assertThat(number).isEqualTo(0);
    assertThat(totalPages).isEqualTo(2);
    assertThat(first).isTrue();
    assertThat(hasNext).isTrue();

    List<Member> content1 = slice.getContent();
//        long totalElements = slice.getTotalElements();
    int number1 = slice.getNumber();
//        int totalPages = slice.getTotalPages();
    boolean first1 = slice.isFirst();
    boolean hasNext1 = slice.hasNext();

    assertThat(content1.size()).isEqualTo(3);
    assertThat(number1).isEqualTo(0);
    assertThat(first1).isTrue();
    assertThat(hasNext1).isTrue();

    // 사용 불가
//        List<Member> content1 = listByAge.getContent();
//        long totalElements = listByAge.getTotalElements();
//        int number1 = listByAge.getNumber();
//        int totalPages = listByAge.getTotalPages();
//        boolean first1 = listByAge.isFirst();
//        boolean hasNext1 = listByAge.hasNext();

    for (Member member : content) {
        System.out.println("member = " + member);
    }
    System.out.println("totalElements = " + totalElements);
}
```
* __Page<>__
  + Page<> 반환 타입은 페이징이 필요하고 전체 데이터의 row 수나 페이지 수가 얼만지 알아야 할 때 사용한다.
  + 전체 데이터의 row 수나 페이지 수를 알기 위해 count 쿼리를 추가로 날린다.
  + @Query 어노테이션을 이용해 count 쿼리를 최적화 하기 위한 방법을 제공한다.(최적화 예시 : select count(m) from Member m left join m.team t where m.age = :age -> select count(m) from Member m where m.age = :age)
  + 일반적으로 Pageable(인터페이스)을 파라미터로 받고 PageRequest(구현체)를 넣는다.
  + getContent(), getTotalElements(), getNumber(), getTotalPages(), isFirst(), hasNext(), map() 등 여러 편의 메서드와 페이징 관련 메서드를 제공한다.
* __Slice<>__
  + 페이징이 필요없고 그냥 데이터만 일정크기로 잘라서 가져오고 싶을 때 사용한다.
  + 페이징 관련 기능을 사용하지 않기 때문에 count쿼리를 날리지 않는다. 그렇기 때문에 페이징 관련 기능을 사용하지 않는다면 Slice를 사용하는 것이 성능상 더 좋다.
  + getTotalElements(), getTotalPages() 등 페이징과 관련된 일부 메서드를 사용하지 못한다.
  + Slice 는 전체 row 수가 몇개인지, 현재 내가 몇 페이지에 있는지 모르기 때문에 만약 limit 를 PageRequest 에 주어진대로 가져오면 더 이상 데이터가 없어도 쿼리를 보내야지 알 수 있다. 따라서 Slice 는 PageRequest 에 주어진 limit 보다 1 크게 데이터를 가져와서 쿼리를 보내지 않아도 다음 페이지가 있는지 알 수 있게 한다.
* __List<>__
  + 페이징이 필요없고 그냥 데이터만 일정크기로 잘라서 가져오고 싶을 때 사용한다.
  + 데이터만 잘라서 가져오고 페이징 관련 메서드는 사용하지 못한다.
## 벌크성 수정 쿼리
```java
public interface MemberRepository extends JpaRepository<Member, Long> {

    @Modifying(clearAutomatically = true)
    @Query("update Member m set m.age = m.age + 1 where m.age >= :age")
    int bulkAgePlus(@Param("age") int age);
}
```
* @Modifying 이 있어야 executeUpdate()를 실행한다. 없으면 getResultList()나 getSingleResult()를 실행한다.
* 벌크성 쿼리 후 추가적인 작업을 한다면 clearAutomatically = true 를 설정해두자(벌크성 쿼리 후 em.clear()해줌)
## 엔티티 그래프
```java
public interface MemberRepository extends JpaRepository<Member, Long> {

    @Override
    @EntityGraph(attributePaths = {"team"})
    List<Member> findAll();
}
```
* attributePaths 에 들어간 값을 페치조인 해준다.(left outer join 이고 다른 조인으로 바꿀 수 없음)
## DynamicUpdate
```java
@Entity
@Getter
@NoArgsConstructor(access = AccessLevel.PROTECTED)
@DynamicUpdate
public class Item extends DateBaseEntity {

    @Id
    @GeneratedValue
    @Column(name = "item_id")
    private Long id;

    @Column(nullable = false, length = 50)
    private String itemName;

    @Column(nullable = false)
    private int price;

    @Column(nullable = false)
    private int stockQuantity;

    @Enumerated(EnumType.STRING)
    private ItemStatus status;

    public void addStock(int stockQuantity) {
        this.stockQuantity += stockQuantity;
    }

    public void removeStock(int orderNumber) {
        int restStock = this.stockQuantity - orderNumber;
        if (restStock < 0) {
            throw new OutOfStockException("재고가 부족합니다.(현재 재고량 : " + this.stockQuantity + ")");
        } else if (restStock == 0) {
            this.status = ItemStatus.SOLD_OUT;
        }
        this.stockQuantity = restStock;
    }
}
```
* JPA는 기본적으로 변경을 감지하여 update 쿼리를 날리게되면 수정된 컬럼만 수정하는 쿼리문을 날리는 것이 아닌 전체 컬럼을 수정하는 쿼리문을 날리게 된다. 하지만 @DynamicUpdate를 사용하면 실제 값이 변경된 컬럼으로만 update 쿼리를 만드는 기능이다. 얼핏 보면 @DynamicUpdate를 사용하는 것이 더 좋아보이지만 default가 아닌 이유는 JPA는 어플리케이션이 처음 로드될 때 엔티티를 전부 스캔하여 업데이트할 쿼리를 캐시 해놓고 사용한다. 하지만 @DynamicUpdate를 사용하면 캐시 된 쿼리를 사용하지 못하고 update마다 새로 동적 쿼리를 생성하고, 이 과정에서 오버헤드가 발생해 오히려 성능이 떨어질 수 있다. 따라서 @DynamicUpdate는 엔티티의 필드 개수가 많고 그중 업데이트되는 필드가 적을 때는 유용하다. 
## 쿼리 힌트
```java
public interface MemberRepository extends JpaRepository<Member, Long> {

    @QueryHints(
        {@QueryHint(name = "org.hibernate.readOnly", value = "true"),
        @QueryHint(name = "javax.persistence.lock.scope", value = "EXTENDED"),
        @QueryHint(name = "javax.persistence.lock.timeout", value ="10000")}
        )
    Member findReadOnlyByUsername(String username);
}
```
* JPA 구현체에게 쿼리 힌트를 전달한다.
* @QueryHint(name = "org.hibernate.readOnly", value = "true") 의 경우에는 스냅샷을 만들지 않아 더티 체킹이 불가능하게 한다.
* @QueryHint(name = "javax.persistence.lock.scope", value = "EXTENDED") 의 경우에는 pessimistic lock scope를 지정할 수 있다.
* @QueryHint(name = "javax.persistence.lock.timeout", value ="10000") 의 경우에는 lock timeout을 지정할 수 있다.
* 이외에도 여러 쿼리 힌트를 지정할 수 있다.
## Lock
```java
public interface TeamRepository extends JpaRepository<Team, Long> {

    @EntityGraph(attributePaths = {"members"})
    // @Lock(LockModeType.NONE)
    // @Lock(LockModeType.OPTIMISTIC)
    // @Lock(LockModeType.OPTIMISTIC_FORCE_INCREMENT)
    // @Lock(LockModeType.PESSIMISTIC_READ)
    // @Lock(LockModeType.PESSIMISTIC_WRITE)
    @Lock(LockModeType.PESSIMISTIC_FORCE_INCREMENT)
    @QueryHints(
        {@QueryHint(name = "javax.persistence.lock.scope", value = "EXTENDED"),
        @QueryHint(name = "javax.persistence.lock.timeout", value ="10000")}
        )
    List<Team> findLockByName(String name);
}
```
```java
@Test
@Rollback(value = false)
public void lock() {

    Team team = new Team("teamA");
    teamRepository.save(team);
    Member member1 = new Member("member1", 10, team);
    memberRepository.save(member1);
    team.setName("teamB");
    // LockModeType.NONE, LockModeType.OPTIMISTIC, LockModeType.OPTIMISTIC_FORCE_INCREMENT 사용 시 update 쿼리에서 version+=1이 된다.(version : 0 -> 1)
    em.flush();
    em.clear();

    Team findTeam = teamRepository.findLockByName("teamB").get(0); // version : 1
    System.out.println(" ============================================== ");
    System.out.println("findTeam = " + findTeam);
    System.out.println(" ============================================== ");

    Member member2 = new Member("member2", 10, team);
    memberRepository.save(member2);
    em.flush(); // version : 1
    em.clear();
    Team findTeam2 = teamRepository.findLockByName("teamB").get(0);
    System.out.println(" ============================================== ");
    System.out.println("findTeam = " + findTeam2);
    System.out.println(" ============================================== ");
    // LockModeType.OPTIMISTIC, LockModeType.OPTIMISTIC_FORCE_INCREMENT 사용 시 트랜잭션 커밋 시점에 version을 select 하는 쿼리를 날려 트랜잭션 시작시점의 version과 비교하여 충돌이 났는지 비교한다.(version : 1)
    //LockModeType.OPTIMISTIC_FORCE_INCREMENT 사용 시 트랜잭션 커밋 시점에 version을 +1하는 update쿼리를 날린다.(version : 1 -> 2)
}
```
* Optimistic Lock : LockModeType.NONE, LockModeType.OPTIMISTIC, LockModeType.OPTIMISTIC_FORCE_INCREMENT
* Pessimistic Lock : LockModeType.PESSIMISTIC_READ, LockModeType.PESSIMISTIC_WRITE, LockModeType.PESSIMISTIC_FORCE_INCREMENT
* __Optimistic Lock__
  + 데이터 갱신시 경합이 발생하지 않을 것이라고 낙관적으로 보고 잠금을 거는 기법이다. 예를 들어 회원정보에 대한 갱신은 보통 해당 회원에 의해서 이루어지므로 동시에 여러 요청이 발생할 가능성이 낮은데 이러한 경우에 Optimistic Lock을 사용한다. 
  + @Version 어노테이션을 이용해 Lock은 엔티티에 버전 정보를 추가해 어플리케이션 레벨에서 Lock을 건다.
* __Pessimistic Lock__ 
  + 동일한 데이터를 동시에 수정할 가능성이 높다는 비관적인 전제로 잠금을 거는 방식이다. 
  + 성능적인 측면에서는 Optimistic Lock보다 좋지 않다.(Lock이 되어있는 동안 다른 트랜잭션가 해당 데이터에 접근하는 것을 블로킹 하기 때문)
  + 데이터베이스 레벨에서 Lock을 건다. 
  + 교착상태에 빠질 가능성이 있다. 그러므로  @QueryHint(name = "javax.persistence.lock.timeout", value ="10000")를 통해 일정시간 이상 대기하지 않도록 timeout을 설정할 수 있다.
  + @QueryHint(name = "javax.persistence.lock.scope", value = "EXTENDED")를 지정하면 연관된 엔티티 객체에 대해서도 Lock이 걸린다.(default : PessimisticLockScope.NORMAL)
* __@Version__
  + @Version은 long, Long, int, Integer, Short, short, Timestamp에만 적용 가능하다.
  + update 쿼리, 트랜잭션 커밋 시점(LockModeType.OPTIMISTIC, LockModeType.OPTIMISTIC_FORCE_INCREMENT)의 version과 데이터 조회 시의 version이 다르면 OptimisticLockException이 발생한다.
* __LockModeType.NONE__
  + 엔티티에 @Version이 적용된 필드가 있으면 자동으로 LockModeType.NONE이 모든 메서드에 적용된다.
  + LockModeType.NONE은 조회 시점부터 수정시점 까지만을 보장한다. 왜냐하면 LockModeType.NONE은 트랜잭션 커밋 시점에 version을 select하는 쿼리를 보내지 않기 때문이다.
* __LockModeType.OPTIMISTIC__
  + 조회 시점부터 트랜잭션이 끝날 때 까지를 보장한다. 
  + 트랜잭션 커밋 시점에 version을 select하는 쿼리를 보내 해당 데이터와 조회 시점의 version 데이터가 같은지를 비교하고 만약 같지 않다면 OptimisticLockException을 발생시킨다.
* __LockModeType.OPTIMISTIC_FORCE_INCREMENT__
  + 트랜잭션 커밋 시점에 version을 select하는 쿼리를 보내 해당 데이터와 조회 시점의 version 데이터가 같은지를 비교하고 만약 같지 않다면 OptimisticLockException을 발생시킨다. 그리고 마지막으로 해당 엔티티의 version을 +1시키는 update쿼리를 날린다.
  + 용도 : 논리적 단위의 엔티티 묶음을 관리할 수 있다. Team과 Member는 일대다 연관관계 이기 때문에 Team객체는 Member객체로 이루어진 List를 가지고 있는데 해당 Team에 새로운 Member가 add() 되어도 List\<Member> members 자체는 변경되지 않았기 때문에 version을 증가하지 않는다. 이러한 변경을 감지하기 위해서 사용하는 것이 LockModeType.OPTIMISTIC_FORCE_INCREMENT이다.
  + Team 객체를 조회하면 해당 Team 객체 뿐만 아니라 Team 객체의 List\<Member> members 에 속하는 모든 Member 객체의 version이 +1 되는 update 쿼리를 날린다. 그렇기 때문에 LockModeType.OPTIMISTIC_FORCE_INCREMENT를 사용한다면 Team 엔티티 뿐만 아니라 Member 엔티티에도 @Version을 적용한 필드가 있어야 한다.(없으면 컴파일 에러)
* __LockModeType.PESSIMISTIC_READ__
  + 조회하는 데이터 row에 S-Lock을 건다.(DB마다 다르지만 select를 한다고해서 S-Lock을 얻는 것은 아니다. S-Lock을 걸지 않고 select를 할 수도 있다. 만약 S-Lock을 무조건 얻고 싶다면 select 쿼리 뒤에 lock in share mode를 덧붙이면 된다(select * from member lock in share mode))
  + 사용하는 데이터베이스가 PESSIMISTIC_READ을 지원하지 않으면 PESSIMISTIC_WRITE로 동작한다.
  + 잘사용하지 않는다.
* __LockModeType.PESSIMISTIC_WRITE__
  + 조회하는 데이터 row에 for update문을 사용하여 X-Lock을 건다.
  + 여기서 X-Lock을 걸었다고 해서 다른 트랜잭션에서 select를 할 수 없는 것이 아니라 X-Lock이 걸리면 해당 row에 대해 S-Lock을 얻지 못하는 것이다. 만약 select를 할 때 S-Lock을 자동으로 거는 DB라면 select을 못하겠지만 S-Lock을 걸지 않고 select를 수행하는 DB의 경우에는 X-Lock이 걸려있더라도 select문을 수행할 수 있다.
  + NON-REPEATABLE READ를 방지한다. 원래 update 쿼리를 날릴 때는 자동으로 X-Lock을 걸지만 select 쿼리에는 X-Lock을 걸지 않는다. 하지만 for update문을 사용하면 select 쿼리라 하더라도 X-Lock을 걸어서 다른 트랜잭션이 데이터를 변경할 수 없다.
  + 매우 중요한(돈 관련 같은) 트랜잭션 수행 시 사용하면 좋다.
* __LockModeType.PESSIMISTIC_FORCE_INCREMENT__
  + @Version이 적용된 필드가 필요하다.
  + 트랜잭션 커밋 시점에 @Version이 적용된 필드를 증가시킨다.
  + 하이버네이트의 경우 nowait를 지원하는 데이터베이스에 대해 for update nowait 옵션을 적용하여 X-Lock을 건다.(Lock을 획득하기 위해 대기하지 않음)