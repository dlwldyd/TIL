# 즉시 로딩과 지연 로딩
## 프록시
<center><img src="./../img/jpa-proxy.png"></center>

```java
Member member = new Member();
entityManager.persist(member);

entityManager.flush();
entityManager.clear();

Member findMember = entityManager.find(Member.class, member.getId()); // 이 시점에 데이터베이스에 쿼리를 보내 데이터를 받아온다.
System.out.println(findMember.getName());
```
```java
Member member = new Member();
entityManager.persist(member);

entityManager.flush();
entityManager.clear();

Member findMember = entityManager.find(Member.class, member.getId()); // findMember에는 실제 엔티티가 아닌 프록시가 들어간다.
System.out.println(findMember.getName());// 이 시점에 데이터베이스에 쿼리를 보내 데이터를 받아오고 엔티티를 생성해 프록시에 있는 target이 실제 엔티티를 가리키도록 한다.
```
* getReference를 통해 데이터를 조회하려 하면 해당 시점에는 쿼리가 나가지 않고 조회하려는 엔티티를 상속한 프록시가 할당되고 영속성 컨텍스트에도 프록시가 등록된다.
* 엔티티에 속한 데이터를 사용할 때 영속성 컨텍스트를 통해 프록시를 초기화(엔티티를 생성해 target이 실제 엔티티를 가리키도록 함)한다. 따라서 getReference로 프록시를 할당받고 detach한 후 엔티티에 속한 데이터를 사용하려 하면 영속성 컨텍스트를 통해 프록시를 초기화 할 수 없기 때문에 예외가 발생한다.(영속성 컨텍스트에 프록시 정보가 사라지기 때문)
* EntityManagerFactory를 통해 PersistenceUnitUtil를 생성하여 isLoaded() 메서드를 사용하면 프록시가 초기화 됐는지를 알 수 있다.
## 지연 로딩
```java
@Entity
public class Member {

    @Id
    @GeneratedValue
    @Column(name = "MEMBER_ID")
    private Long id;

    @Column(name = "USERNAME")
    private String name;

    @ManyToOne(fetch = FetchType.LAZY) 
    @JoinColumn(name = "TEAM_ID")
    private Team team;

    ...

}
```
* fetch 속성을 FetchType.LAZY로 하면 데이터베이스에서 Member 엔티티를 조회할 때 Team 엔티티도 가져오는게 아니라 Team에는 프록시를 넣어두가 Team 엔티티가 쓰일 때 데이터베이스에 쿼리를 날려 Team 엔티티를 가져온다.
* @OneToMany, @ManyToMany는 기본이 지연 로딩으로 설정되어 있다.
* 거의 모든 경우에 FetchType.LAZY를 사용해야 한다.
## 즉시 로딩
```java
@Entity
public class Member {

    @Id
    @GeneratedValue
    @Column(name = "MEMBER_ID")
    private Long id;

    @Column(name = "USERNAME")
    private String name;

    @ManyToOne(fetch = FetchType.EAGER) 
    @JoinColumn(name = "TEAM_ID")
    private Team team;

    ...

}
```
* fetch 속성을 FetchType.EAGER로 하면 데이터베이스에서 TEAM 테이블을 조인해서 쿼리하여 Team엔티티도 한꺼번에 가져온다.
* @ManyToOne, @OneToOne은 기본이 즉시 로딩으로 설정되어 있다.
* 거의 사용하지 않는다.
  + 참조 하려는 엔티티가 많은 다른 엔티티를 참조하고 있다면 예상하지 못한 쿼리가 발생할 수 있다.
  + JPQL에서 N+1 문제가 발생할 수 있다.
  + 만약 즉시 로딩처럼 한번에 데이터를 조회할 필요가 있다면 JPQL fetch 조인이나 엔티티 그래프를 활용하는게 좋다.(대안이 있다.)