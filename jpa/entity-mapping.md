# 엔티티 매핑
## @Entity
```java
@Entity(name = "hello")
public class Member {
    ...
}
```
* @Entity가 붙은 클래스는 JPA가 관리하고, 엔티티라 부른다. JPA를 사용해서 테이블과 매핑할 클래스는 @Entity 필수이다.
* 기본 생성자가(파라미터가 없는 public 또는 protected 생성자) 필수이다.
* final 클래스, enum, interface, inner 클래스에 사용할 수 없다.
* 필드에 final을 사용할 수 없다.
* 속성
  + name : JPA에서 사용하는 엔티티의 이름을 지정할 수 있다.(기본값 : 클래스 이름을 그대로 사용)
## @Table
```java
@Entity
@Table(name = "user")
public class Member {
    ...
}
```
* 엔티티와 매핑할 테이블의 정보를 지정할 수 있다.
* 속성
  + name : 매핑할 테이블 이름을 지정할 수 있다.
  + uniqueContraints : DDL생성시 유니크 제약조건을 생성할 수 있다.
## @Id
```java
@Id
private Long id;
```
* 기본키로 매핑한다. 기본키를 직접 할당할 때에는 @Id만 사용하면 된다.
## @@GeneratedValue
```java
@Id
@GeneratedValue(strategy = GenerationType.AUTO)
private Long id;
```
* 기본키를 직접 할당하지 않고 자동으로 생성시키려면 @GeneratedValue를 사용해야 한다.
* 속성
  + strategy : 기본키 생성 전략을 정한다.
    - GenerationType.AUTO : db 방언에 맞춰서 기본키를 생성한다.(기본값)
    - GenerationType.IDENTITY : 기본키 생성을 데이터베이스에 위임한다.
    - GenerationType.SEQUENCE : sequence 객체를 통해 기본키 생성
    - GenerationType.TABLE : 키 생성 전용 테이블을 만들어서 SEQUENCE 전략처럼 사용(데이터베이스에 따라 SEQUENCE 를 사용하지 못할 수 있기 때문에)
  + generator : SequenceGenerator 나 TableGenerator 를 지정할 수 있음(SequenceGenerator 나 TableGenerator 사용 시 필수)
## @SequenceGenerator
```java
@Entity
@SequenceGenerator(
        name = "MEMBER_SEQ_GENERATOR", 
        sequenceName = "MEMBER_SEQ", 
        initialValue = 1, allocationSize = 2) 
public class Member {
    ...
}
```
* SequeceGenerator 생성한다.(직접 데이터베이스에서 create 할 수도 있음)
* 속성
  + name : SequenceGenerator 이름(JPA에서 사용)
  + sequenceName : 매핑할 데이터베이스 SequenceGenerator 이름(데이터베이스에서의 이름)
  + initialValue : 시퀀스 초기값(기본값 1)
  + allocationSize : 처음 persist 를 하면 키 값이 없기 때문에 영속 상태로 만들 수가 없다. 따라서 키 값을 데이터베이스에 쿼리를 보내서 받아와야한다. 이 때 쿼리를 보내 테이블(시퀀스)로 부터 값을 받아 오는데 만약 persist 를 할 때마다 쿼리를 보내면 성능상 좋지 않을 것이다. 따라서 만약 allocationSize = 50 일 때 쿼리를 보내면 테이블(시퀀스)의 값(MEMBER_SEQ, MEMBER_SEQ)은 50증가하고 영속석 컨테이너에서는 다음 50개의 persist 까지는 데이터베이스에 쿼리를 보내지 않고 50개까지 키를 할당할 수 있다. 하지만 allocationSize 를 너무 크게 잡으면 만약 어플리케이션을 종료했을 때 많게는 allocationSize 만큼 키값의 공백이 생겨날 수 있다.(쿼리를 보내고 allocationSize 중 얼만큼을 사용했는지는 메모리에 저장되어 있기 때문) 그렇기 때문에 allocationSize 를 너무 크게 잡는 것도 좋지 않다.
## @TableGenerator
```java
@Entity
@TableGenerator( 
        name = "MEMBER_SEQ_GENERATOR", 
        table = "MY_SEQUENCES", 
        pkColumnValue = "MEMBER_SEQ", allocationSize = 2)
public class Member {
    ...
}
```
* TableGenerator(키 생성 전용 테이블) 생성(직접 데이터베이스에서 create 할 수도 있음)(잘 안씀)
* 속성
  + name : TableGenerator 이름(JPA에서 사용)
  + table : TableGenerator(키 생성 전용 테이블) 이름
  + pkColumnValue : 기본키 값을 지정(컬럼 이름이 아니라 데이터로 들어가는 값)
  + allocationSize : @SequenceGenerator의 allocationSize와 같다.
## @Column
```java
@Column(name = "name")
private String username;
```
* 컬럼에 대한 정보를 지정할 때 사용한다.
* 속성
  + name : 컬럼 이름을 지정할 수 있다.
  + insertable : 엔티티를 insert 할 때 해당 필드는 insert 하지 않는다(db 에서 설정된 default 값이 들어간다)(기본값 true)(읽기 전용일 때 사용)
  + updatable : 엔티티의 해당 필드가 바뀌면 update 쿼리를 db에 날릴지 결정(기본값 true)(읽기 전용일 때 사용)
  + nullable : 해당 필드에 null 을 넣을 수 있는지 결정(기본값 true)
  + length : varchar 길이 결정(기본값 255)(String 타입에서 사용)
  + columnDefinition : 컬럼 정보를 직접 넣을 수 있다.(예시 : columnDefinition =  "varchar(100) default 'EMPTY'")
  + precision : BigDecimal 타입이나 BigInteger 타입에서 사용한다. 숫자의 자리수를 정한다. 소수점을 포함한 자리수이다. scale 과 함께 사용한다.
  + scale : BigDecimal 타입이나 BigInteger 타입에서 사용한다. 소수점의 자리수를 정한다. precision 과 함께 사용한다.(예시 : precision=19, scale=8)
## @Enumerated
```java
public enum RoleType {
    USER, ADMIN
}
```
```java
@Enumerated(EnumType.STRING)
private RoleType roleType;
```
* enum 타입을 쓸 때 붙인다. db 에는 enum 과 같은 타입이 없기 때문(기본값 EnumType.ORDINAL)
* 속성
  + value : 어떤 값을 테이블에 저장할지 지정(속성이 이것밖에 없음)
    - EnumType.ORDINAL : enum 의 순서를 db에 저장(USER=0, ADMIN=1에서 1, 2를 저장) -> 사용 안함
    - EnumType.STRING : enum 의 이름을 db에 저장(USER=0, ADMIN=1에서 USER, ADMIN 을 저장)
## @Temporal
```java
@Temporal(TemporalType.TIMESTAMP)
private Date createdDate;
```
* Date 타입을 사용할 때 붙인다. 자바에는 Date 안에 날짜와 시간이 전부 들어가 있지만 데이터베이스에는 날짜만 사용하거나 시간만 사용하거나 둘 다 사용할 수 있다.
* 자바8 부터는 Date 대신 LocalDate, LocalTime, LocalDateTime 을 쓰면 어노테이션 안붙여도 자동으로 매핑된다.
* 속성
  + value : 테이블의 속성 지정(속성이 이것밖에 없다)
    - TemporalType.DATE : 날짜만 사용(LocalDate)
    - TemporalType.TIME : 시간만 사용(LocalTime)
    - TemporalType.TIMESTAMP : 날짜와 시간 둘다 사용(LocalDateTime)
## @Lob
```java
@Lob
private String description;
```
* VARCHAR 를 넘어서는 큰 데이터를 넘기고 싶을 때는 @Lob 을 쓴다.(Large Object)
* LOB 의 종류에는 BLOB(바이너리 데이터), CLOB(텍스트, 보통 4000자를 넘어가는 경우 CLOB 을 사용한다) 등이 있다.
* 어떠한 LOB 이 사용될지는 변수 타입에 의해 결정된다.(문자면 CLOB, 아니면 BLOB)
## @Transient
```java
@Transient
private int tmp;
```
* @Transient를 붙이면 해당 필드는 테이블에 매핑되지 않는다.