# 상속관계 매핑, @MappedSuperClass
## 상속관계 매핑
### 개별타입 기준 테이블 변환
<center><img src="./../img/joined.png"></center>

```java
@Entity
@Inheritance(strategy = InheritanceType.JOINED)
//@DiscriminatorColumn(name = "DTYPE")
public abstract class Item {

    @Id
    @GeneratedValue
    @Column(name = "ITEM_ID")
    private Long id;

    private String name;

    private int price;

    ...

}
```
```java
@Entity
//@DiscriminatorValue("ALBUM")
public class Album extends Item {

    private String artist;

    ...

}
```
```java
@Entity
//@DiscriminatorValue("BOOK")
public class Book extends Item {

    private String author;

    private String isbn;

    ...
    
}
```
```java
@Entity
//@DiscriminatorValue("MOVIE")
public class Movie extends Item {

    private String director;

    private String actor;

    ...

}
```
* @Inheritance(strategy = InheritanceType.JOINED)를 사용하면 개별타입 기준으로 엔티티를 테이블로 변환해 매핑한다.
* 가장 많이 사용하는 전략이다.
* 장점
  + 테이블을 정규화할 수 있다.
  + 저장공간을 효율화 할 수 있다.
* 단점
  + 조회시 조인을 사용하기 때문에 성능이 저하될 수 있다.(많이 저하되지는 않음)
  + 슈퍼타입과 서브타입에 데이터를 저장해야 하기 때문에 insert쿼리시 쿼리가 2번 호출된다.
* @DiscriminatorColumn : 해당 row 가 어떤 서브타입에 속하는지 나타내는 컬럼이 추가된다. 왠만하면 사용하는 것이 좋다.(기본값 : DTYPE)
* JOINED 전략에서는 @DiscriminatorColumn 을 안쓰면 DTYPE이 추가가 안된다.
* @DiscriminatorValue : 슈퍼 타입의 row 가 어떤 서브타입에 해당하는 지 나타내는 컬럼(DTYPE)에 들어갈 값을 지정한다.(기본값 : 엔티티 이름)
### 슈퍼타입 기준 테이블 변환
<center><img src="./../img/single-table.png"></center>

```java
@Entity
@Inheritance(strategy = InheritanceType.SINGLE_TABLE)
public abstract class Item {

    ...

}
```
* @Inheritance(strategy = InheritanceType.SINGLE_TABLE)을 사용하면 슈퍼타입 기준으로 엔티티를 테이블로 변환해 매핑한다. 나머지 구조를 변경할 필요없이 전략만 바꾸면 된다.
* 장점
  + 조회시 조인을 사용하지 않기 때문에 조회 성능이 좋다.
* 단점
  + 자식 엔티티가 매핑한 컬럼은 모두 null을 허용해야 한다.
  + 단일 테이블에 모두 저장하기 때문에 테이블이 커질 수 있고, 테이블이 너무 커지게 되면 InheritanceType.JOINED보다 조회 성능이 안좋아질 수 있다.
* @DiscriminatorColumn : SINGLE_TABLE 전략에서는 @DiscriminatorColumn 을 안써도 자동으로 DTYPE이 추가된다.(default 값으로)
### 서브타입 기준 테이블 변환
<center><img src="./../img/table-per-class.png"></center>

```java
@Entity
@Inheritance(strategy = InheritanceType.TABLE_PER_CLASS)
public abstract class Item {

    ...

}
```
* @Inheritance(strategy = InheritanceType.TABLE_PER_CLASS)를 사용하면 서브타입 기준으로 엔티티를 테이블로 변환해 매핑한다. 나머지 구조를 변경할 필요없이 전략만 바꾸면 된다.
* @DiscriminatorColumn : TABLE_PER_CLASS 전략에서는 @DiscriminatorColumn 를 써도 의미가 없기 때문에 DTYPE이 추가되지 않는다.
* 사용하지 않는게 좋다.
## @MappedSuperClass
<center><img src="./../img/MappedSuperClass.png"></center>

```java
@MappedSuperclass
public abstract class BaseEntity {

    @Id
    @GeneratedValue
    private Long id;

    //@Column(name = "NAME")
    private String name;

    ...

}
```
```java
@Entity
public class Member extends BaseEntity {

    private String email;

    ...

}
```
```java
@Entity
public class Seller extends BaseEntity {

    private String shopName;

    ...

}
```
* 여러 엔티티가 공통으로 사용하는 필드를 한 곳에 모을 때 사용한다.
* 데이터베이스 조회시 @MappedSuperclass가 붙은 부모 타입으로는 조회가 불가능 하다.(entityManager.find(BaseEntity.class, member.getId())) 
* 직접 생성해서 사용할 일은 없으므로 추상클래스로 만드는 것이 좋다.
* 엔티티가 상속을 받을 때는 @Entity가 붙은 클래스나 @MappedSuperclass가 붙은 클래스만 상속을 받을 수 있다.