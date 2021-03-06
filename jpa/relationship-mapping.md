# 연관관계 매핑
## N:1 연관관계
<center><img src="./../img/n-1relationship.png"></center>

```java
@Entity
public class Member {

    @Id
    @GeneratedValue
    @Column(name = "MEMBER_ID")
    private Long id;

    @ManyToOne
    @JoinColumn(name = "TEAM_ID")
    private Team team;

    @Column(name = "USERNAME")
    private String username;

    ...

}
```
```java
@Entity
public class Team {

    @Id
    @GeneratedValue
    @Column(name = "TEAM_ID")
    private Long id;

    private String name;

    ...

}
```
* 가장 많이 사용하는 연관관계이다.
* 연관관계의 주인은 N에 해당하는 쪽의 객체이다.
* N:1에서 N에 해당하는 쪽의 테이블이 외래키를 가진다.
* @ManyToOne 어노테이션에서 optional 속성을 통해 연관된 엔티티가 항상 있어야 하는지 정할 수 있다.(기본값 : true)
* @Joincolumn 어노테이션에서 외래키가 참조하는 테이블의 기본키가 아닐 경우 referencedColumnName을 통해 외래키가 참조하는 대상 테이블의 컬럼 명을 지정할 수 있다.(기본값 : 참조하는 테이블의 기본키 컬럼명)
## 1:N 연관관계
<center><img src="./../img/1-nrelationship.png"></center>

```java
@Entity
public class Team {

    @Id
    @GeneratedValue
    @Column(name = "TEAM_ID")
    private Long id;

    private String name;

    @OneToMany
    @JoinColumn(name = "TEAM_ID")
    private List<Member2> members = new ArrayList<>();

    ...

}
```
```java
@Entity
public class Member {

    @Id
    @GeneratedValue
    @Column(name = "MEMBER_ID")
    private Long id;

    @Column(name = "USERNAME")
    private String username;

    ...

}
```
* 연관관계의 주인은 1에 해당하는 쪽의 객체이다.
* 1:N에서 N에 해당하는 쪽의 테이블이 외래키를 가진다.
* 1:N연관관계를 사용할 시 @JoinColumn을 사용하지 않으면 조인테이블을 생성하기 때문에 @JoinColumn을 사용해야한다.
* 잘 사용하지 않는다.
## 1:1 연관관계
<center><img src="./../img/1-1relationship.png"></center>

```java
@Entity
public class Member {

    @Id
    @GeneratedValue
    @Column(name = "MEMBER_ID")
    private Long id;

    @OneToOne
    @JoinColumn(name = "LOCKER_ID")
    private Locker locker;

    @Column(name = "USERNAME")
    private String username;

    ...

}
```
```java
@Entity
public class Locker {

    @Id
    @GeneratedValue
    @Column(name = "LOCKER_ID")
    private Long id;

    private String  name;

    ...

}
```
* 1:1 연관관계에서는 테이블을 생성할 때 외래키에 유니크 제약조건이 추가되야 한다.
## N:M 연관관계
<center><img src="./../img/n-mrelationship.png"></center>

```java
@Entity
public class Member {

    @Id
    @GeneratedValue
    @Column(name = "MEMBER_ID")
    private Long id;

    @OneToMany(mappedBy = "member")
    private List<MemberProduct> memberProducts = new ArrayList<>();

    @Column(name = "USERNAME")
    private String username;

    ...

}
```
```java
@Entity
@Table(name = "ORDER")
public class MemberProduct {

    @Id
    @GeneratedValue
    @Column(name = "ORDER_ID")
    private Long id;

    @ManyToOne
    @JoinColumn(name = "MEMBER_ID")
    private Member member;

    @ManyToOne
    @JoinColumn(name = "PRODUCT_ID")
    private Product product;

    private Integer orderAmount;

    private LocalDate orderDate;

    ...

}
```
```java
@Entity
public class Product {

    @Id
    @GeneratedValue
    @Column(name = "PRODUCT_ID")
    private Long id;

    private String name;

    ...

}
```
* @ManyToMany를 사용하여 N:M 연관관계 매핑을 할 수 있지만 여러가지 제약사항이 많기 때문에(연결 테이블에 필드 추가 못함 등) 잘 사용하지 않는다.
* 연결 테이블용 엔티티를 만들어서 N:1 연관관계 매핑을 통해 N:M 연관관계를 매핑한다.
## 양방향 연관관계
<center><img src="./../img/bothways-relationship.png"></center>

```java
@Entity
public class Member {

    @Id
    @GeneratedValue
    @Column(name = "MEMBER_ID")
    private Long id;

    @ManyToOne
    @JoinColumn(name = "TEAM_ID")
    private Team team;

    @Column(name = "USERNAME")
    private String username;

    ...

}
```
```java
@Entity
public class Team {

    @Id
    @GeneratedValue
    @Column(name = "TEAM_ID")
    private Long id;

    @OneToMany(mappedBy = "team")
    private List<Member> members = new ArrayList<>();

    private String name;

    ...

}
```
* mappedBy 속성을 통해 양방향 연관관계를 만들 수 있다. 단방향 연관관계를 만든 상태에서 양방향 연관관계를 만들면 데이터베이스에는 아무런 변화도 없이 객체 간에만 변화가 있기 때문에 단방향 연관관계로 설계한 후 필요한 경우에 양방향 연관관계로 만드는 것이 좋다.
* mappedBy : 양방향 연관관계에서 연관관계의 주인을 정한다. 데이터베이스의 경우에는 양방향 연관관계가 TEAM_ID 라는 외래키 하나만으로 성립된다. 하지만 객체에서는 Team 의 경우에는 List\<Members> members, Member 의 경우에는 Team team 에 의해 양방향 연관관계가 성립한다. 따라서 데이터베이스의 경우에는 TEAM_ID 라는 외래키를 바꾸는 것만으로 엔티티간의 연관관계를 수정할 수 있지만, 객체에서는 만약 연관관계의 주인을 정하지 않는다면 List\<Members> members 와 Team team 중 어떤 것을 수정해서 엔티티간의 연관관계를 수정할 수 있는지 알 수 없다. 그렇기 때문에 연관관계의 주인을 정하여 List\<Members> members 와 Team team 중 어떤 것을 수정하였을 때 데이터베이스에 변화가 일어날 지 정해야 한다. 이 때 연관관계의 주인만이 외래키를 등록, 수정할 수 있고 주인이 아닌 쪽은 읽기만 가능하다. 그리고 연관관계의 주인이 아닌 쪽에서의 수정은 객체 상으로는 수정 가능하지만 데이터베이스에 적용할 때 JPA 에서 관련된 쿼리를 내보내지 않는다. 
* 외래키를 수정할 때는 연관관계의 주인만 수정하는게 아니라 양쪽 다 수정하는 것이 좋다. 왜냐하면 연관관계의 주인만 수정하고 연관관계의 주인이 아닌 쪽의 객체를 persist 했을 때, 해당 객체는 db에 저장되는게 아니라 영속성 컨테이너의 1차 캐시에 저장되고 이 때 List\<Member> members 는 비어있다. 그리고 해당 객체를 find 로 찾아보면 List\<Member> members 는 그대로 비어있다. 그렇기 때문에 이러한 영속성 컨테이너로 인한 연관관계의 불일치를 해결하기 위하여 양쪽을 모두 수정하는 것이 좋다.(객체 지향적으로 생각해도 둘 다 수정하는게 맞다.) 이 때 양쪽을 모두 수정하는 편의 메서드를 작성하는 편이 실수로라도 한쪽만 수정하는 일을 줄일 수 있기 때문에 좋다.
* 연관관계의 주인은 mappedBy 를 사용하지 않고, 주인이 아닌 쪽은 mappedBy 속성에 연관관계의 주인의 이름(필드 이름)을 추가해야 한다. 연관관계의 주인을 정할 때는 외래키가 있는 곳(테이블 상으로)(n:1 중 n에 해당)을 연관관계의 주인으로 정하는 것이 좋다.