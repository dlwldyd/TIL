# 영속성 컨텍스트
## 영속성 컨텍스트 생명주기
<center><img src="./../img/entity-lifecycle.png"></center>

## 비영속 상태
```java
Member member = new Member(1L, "hello");
```
* 처음 엔티티를 생성했을 때 해당 엔티티는 비영속 상태에 있다. 준영속 상태와의 차이점은 영속성 컨텍스트에 관리되었던 적이 있느냐 없느냐로, 새로 생성된 엔티티는 한번도 영속성 컨텍스트에 관리되었던 적이 없기 때문에 비영속 상태이다.
## 영속 상태
<center><img src="./../img/entity-persist.png"></center>

* 영속성 컨텍스트에는 1차 캐시가 있다. 이러한 1차 캐시에 엔티티가 저장되면 해당 엔티티가 영속상태에 있다고 한다.
```java
Member member = new Member(1L, "hello");
entityManager.persist(member);
```
* 엔티티를 영속상태로 만든다. 그리고 쓰기 SQL 저장소에 insert 쿼리를 생성 후 저장한다.(db에 쿼리를 보내지는 않는다)
* 기본키 생성 전략을 GenerationType.IDENTITY로 하면 데이터베이스에 쿼리를 날려야 기본키 값을 알 수 있기 때문에 기본키 생성 전략이 GenerationType.IDENTITY일 경우에는 바로 insert 쿼리를 db에 날린다.
```java
Member findMember = entityManager.find(Member.class, 1L);
```
* 만약 1차 캐시에 원하는 엔티티가 없으면(Member객체 중 키가 1L인 엔티티) db에 select 쿼리를 보내 데이터를 받아 해당 엔티티를 영속상태로 만든다. 엔티티가 있으면 쿼리를 보내지 않고 1차 캐시에서 값을 받는다.
* find 뿐만 아니라 데이터베이스를 조회하는 JPQL 등 데이터베이스를 조회하는 경우에 이렇게 동작한다.
## 준영속 상태
```java
Member member = new Member(1L, "hello");
entityManager.persist(member);
entityManager.detach(member);
```
* 엔티티를 영속상태에서 준영속상태로 만든다.
  1. 1차 캐시에서 해당 엔티티가 제거된다.
  2. 쓰기 지연 SQL 저장소에 저장되었던 관련 SQL 모두 제거된다. 따라서 flush가 일어나도 데이터베이스에 저장이 되지 않는다.
```java
entityManager.clear();
```
* 영속성 컨텍스트 전체를 준영속 상태로 만듦(영속성 컨텍스트를 완전히 초기화)
```java
entityManager.close();
```
* entityManager를 close 할 때도 영속성 컨텍스트가 완전히 초기화된다.