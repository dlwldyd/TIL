# JPA 기본
## 설정
### 스프링 부트 사용
```properties
#application.properties
#필수
spring.datasource.url=jdbc:h2:tcp://localhost/~/test
spring.datasource.driver-class-name=org.h2.Driver
spring.datasource.username=sa
spring.datasource.password=
spring.jpa.database-platform=org.hibernate.dialect.H2Dialect

#옵션
spring.jpa.show-sql=true
spring.jpa.hibernate.ddl-auto=none
#spring.jpa.hibernate.ddl-auto=create
#spring.jpa.hibernate.ddl-auto=create-drop
#spring.jpa.hibernate.ddl-auto=update
#spring.jpa.hibernate.ddl-auto=validate
```
* spring.datasource.url=jdbc:h2:tcp://localhost/~/test : 데이터베이스에 접근하기한 url을 명시한다.
* spring.datasource.driver-class-name=org.h2.Driver : 사용하는 데이터베이스의 드라이버를 추가해야한다.
* spring.datasource.username=sa : 데이터베이스에 접근할때의 username을 명시한다.
* spring.datasource.password= : 해당 유저의 패스워드
* spring.jpa.database-platform=org.hibernate.dialect.H2Dialect : jpa를 사용하면서 쿼리를 생성할 때 어떤 데이터베이스에 대한 쿼리를 생성할지 명시해야한다.
* #spring.jpa.hibernate.ddl-auto=create : 기존테이블 삭제 후 다시 생성 (DROP + CREATE), 개발초기에만 사용하고 테스트 서버나 운영 서버에서는 사용하지 않는 것이 좋다.
* #spring.jpa.hibernate.ddl-auto=create-drop : create와 같으나 어플리케이션 종료시점에 테이블 DROP, 개발초기에만 사용하고 테스트 서버나 운영 서버에서는 사용하지 않는 것이 좋다.
* #spring.jpa.hibernate.ddl-auto=update : 변경된 부분만 반영한다. alter 쿼리가 나간다. 운영 서버에서는 사용하지 않는 것이 좋다. 
* #spring.jpa.hibernate.ddl-auto=validate : 엔티티와 테이블이 정상 매핑되었는지만 확인한다.
```yml
# application.yml
spring:
  datasource:
    url: jdbc:h2:tcp://localhost/~/jpashop
    username: sa
    password:
    driver-class-name: org.h2.Driver

  jpa:
    hibernate:
      ddl-auto: create
    properties:
      hibernate:
#        show_sql: true System.out.println 을 통해 실행되는 쿼리 출력
        format_sql: true # 쿼리가 한줄로 나오는게 아니라 보기 편하도록 포매팅 해줌
        default_batch_fetch_size: 100
logging:
  level:
    org.hibernate.SQL: debug # 로거를 통해 실행되는 쿼리 출력
    org.hibernate.type: trace # '?'로 나오는 쿼리파라미터에 어떤게 바인딩 되는지 로그로 찍어줌
```
* application.xml이 아닌 application.yml로도 설정 할 수 있다.
### 스프링 부트 사용X
```xml
<!--persistence.xml-->
<?xml version="1.0" encoding="UTF-8"?>
<persistence version="2.2"
             xmlns="http://xmlns.jcp.org/xml/ns/persistence" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
             xsi:schemaLocation="http://xmlns.jcp.org/xml/ns/persistence http://xmlns.jcp.org/xml/ns/persistence/persistence_2_2.xsd">
    <persistence-unit name="hello">
        <properties>
            <!-- 필수 -->
            <property name="javax.persistence.jdbc.driver" value="org.h2.Driver"/>
            <property name="javax.persistence.jdbc.user" value="sa"/>
            <property name="javax.persistence.jdbc.password" value=""/>
            <property name="javax.persistence.jdbc.url" value="jdbc:h2:tcp://localhost/~/test"/>
            <property name="hibernate.dialect" value="org.hibernate.dialect.H2Dialect"/>

            <!-- 옵션 -->
            <property name="hibernate.show_sql" value="true"/>
            <property name="hibernate.format_sql" value="true"/>
            <property name="hibernate.use_sql_comments" value="true"/>
            <property name="hibernate.hbm2ddl.auto" value="none" />
            <!--<property name="hibernate.hbm2ddl.auto" value="create" />-->
            <!--<property name="hibernate.hbm2ddl.auto" value="create-drop" />-->
            <!--<property name="hibernate.hbm2ddl.auto" value="update" />-->
            <!--<property name="hibernate.hbm2ddl.auto" value="validate" />-->
        </properties>
    </persistence-unit>
</persistence>
```
* persistence.xml은 src/main/resources/META-INF에 위치해야한다.(필수)
* \<persistence-unit name="hello"> : persistence-unit 이름을 정한다. 일반적으로 연결한 데이터베이스 하나당 하나의 persistence-unit을 등록한다.
* \<property name="javax.persistence.jdbc.driver" value="org.h2.Driver"/> : 사용하는 데이터베이스의 드라이버를 추가해야한다.
* \<property name="javax.persistence.jdbc.user" value="sa"/> : 데이터베이스에 접근할때의 username을 명시한다.
* \<property name="javax.persistence.jdbc.password" value=""/> : 해당 유저의 패스워드
* \<property name="javax.persistence.jdbc.url" value="jdbc:h2:tcp://localhost/~/test"/> : 데이터베이스에 접근하기한 url을 명시한다.
* \<property name="hibernate.dialect" value="org.hibernate.dialect.H2Dialect"/> : jpa를 사용하면서 쿼리를 생성할 때 어떤 데이터베이스에 대한 쿼리를 생성할지 명시해야한다.
* create, create-drop, update, validate는 gradle에서와 동작이 같다.
## 기본 동작
### Entity
```java
@Entity
//@Table(name = "USER") : 엔티티가 저장될 테이블 이름을 지정할 수 있다.
public class Member {

    @Id
    private Long id;

    //@Column(name = "username") : 컬럼 이름을 지정할 수 있다.
    private String name;

    //jpa 에서 엔티티로 사용시 기본 생성자 필수
    public Member() {
    }

    public Member(Long id, String name) {
        this.id = id;
        this.name = name;
    }

    public Long getId() {
        return id;
    }

    public void setId(Long id) {
        this.id = id;
    }

    public String getName() {
        return name;
    }

    public void setName(String name) {
        this.name = name;
    }
}
```
* 엔티티로 사용할 객체에는 @Entity 어노테이션을 붙여야 한다.
* 테이블의 이름과 컬럼의 이름을 지정하지 않으면 자동으로 클래스이름과 변수이름과 같은 이름으로 지정되지만 @Table과 @Column 어노테이션으로 이름을 지정할 수 있다.
* JPA에서 엔티티로 사용할 때 기본생성자는 필수적으로 있어야 한다.
### EntityManagerFactory
```java
EntityManagerFactory emf = Persistence.createEntityManagerFactory("hello");
```
* 데이터베이스 하나당 엔티티매니저 팩토리는 하나만 사용한다.
* 스프링을 사용하면 자동화 해준다.
### EntityManager
```java
EntityManager em = emf.createEntityManager();
```
* 한번의 클라이언트의 요청마다 하나의 매니저를 만든다.
* 스프링을 사용하면 자동화해준다.
### EntityTrasection
```java
EntityTransaction tx = em.getTransaction();
tx.begin();
```
* jpa 는 데이터를 변경하는 작업을 할 때(CUD) 트랜잭션 안에서 작업을 해야한다.
* 스프링을 사용하면 자동화 해준다.
### persist
<center><img src="./../img/entity-persist.png" width="70%" height="70%"></center>
<center><img src="./../img/entity-persist2.png" width="70%" height="70%"></center>

```java
Member member = new Member(1L, "hello");
em.persist(member);
```
* db에 저장되는게 아니라 비영속 상태를 영속상태로 만들고 쓰기 지연 SQL 저장소에 insert쿼리를 저장한다.(buffering)
### find
<center><img src="./../img/entity-find.png" width="70%" height="70%"></center>

```java
Member findMember = em.find(Member.class, 1L);
```
* 영속성 컨텍스트에 키가 1L인 Member객체가 있으면 그 값을 반환하고 없으면 데이터베이스에서 해당 데이터를 조회하는 select쿼리를 전달해 데이터를 받아오고 해당 엔티티를 영속상태로 만든 후 그 값을 반환한다.
### remove
```java
Member findMember = em.find(Member.class, 1L);
em.remove(findMember);
```
* 영속성 컨텍스트에서 findMember엔티티를 삭제하고 delete쿼리를 쓰기 지연 SQL 저장소에 저장한다.
### update
<center><img src="./../img/entity-update.png" width="70%" height="70%"></center>

```java
Member findMember = em.find(Member.class, 1L);
findMember.setName("helloJPA");
```
* transaction.commit()시점에 스냅샷과 비교하고 변경이 있으면 update 쿼리를 쓰기 지연 SQL 저장소에 쿼리 생성한다. 스냅샷은 엔티티가 처음 영속상태가 됬을 때의 값이다.
### merge
```java
Member member = new Member(1L, "hello");
em.persist(member);

em.flush();
em.detach(member);

em.merge(member);
```
* em.persist(member)는 persist 하는 객체가 비영속상태에 있다고 가정하기 때문에 persist() 메서드를 호출한 후 flush를 해보면 insert 쿼리만 나간다.
* em.merge(member)는 merge 하는 객체가 준영속상태에 있다고 가정하기 때문에(영속성 컨텍스트에는 없지만 데이터베이스에는 있음) merge() 메서드를 호출한 후 flush를 해보면 select 쿼리를 보낸 후 select 쿼리로 받은 객체를 member 객체로 치환하여 insert(데이터베이스에 데이터가 없으면)나 update(데이터가 있으면) 쿼리를 보낸다. 이 때 update 쿼리를 보낼 때 다른 부분에 대해서만 update 쿼리를 보내는게 아니라 컬럼 값이 같더라도 모든 컬럼을 치환하는 update 쿼리를 보낸다. 
* 데이터베이스에 데이터를 저장할 때 persist()대신에 merge()를 사용하면 select쿼리가 한번 나가기 때문에 비용이 더 비싸다 그렇기 때문에 __merge()는 준영속상태를 영속상태로 만들 때를 제외하고는 쓰지 않는 것이 좋다.__
### JPQL
```java
List<Member> result = em.createQuery("select m from Member as m", Member.class).getResultList();
```
* SQL 은 테이블을 대상으로 쿼리를 하지만 JPQL 은 엔티티 객체를 대상으로 쿼리를 한다.(Member:엔티티, m:별칭(필수))
### detach
```java
Member member = new Member(1L, "hello");
em.persist(member);
em.detach(member);
```
* 영속 상태를 준영속 상태로 만든다. 1차 캐시에 해당 member의 데이터를 삭제하고 그와 관련된 쿼리를 쓰기 지연 SQL 저장소에서 삭제한다.
### commit
<center><img src="./../img/entity-commit.png" width="70%" height="70%"></center>

```java
tx.commit();
```
* 쓰기 지연 SQL 저장소에 저장되어 있던 쿼리를 db에 날리고(flush) 데이터베이스 커밋한다.(롤백 불가)
* 스프링을 사용하면 자동화해준다.
### rollback
```java
tx.rollback();
```
* 트랜잭션을 롤백한다.
* 스프링을 사용하면 자동화해준다.
### close
```java
em.close();
```
* 클라이언트의 데이터베이스 커넥션이 끝날 때 엔티티매니저를 close 한다.
* 영속성 컨텍스트가 관리하던 영속 상태의 엔티티가 모두 준영속 상태가 된다.
* 스프링을 사용하면 자동화해준다.
```java
emf.close();
```
* 어플리케이션이 끝날 때 앤티티매니저 팩토리를 close한다.
* 스프링을 사용하면 자동화해준다.