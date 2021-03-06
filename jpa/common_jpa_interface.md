# 공통 인터페이스
## 기본 동작 방식
<center><img src="./../img/spring-data-jpa-repository.png" height = "70%" width = "70%"></center>

* JpaRepository에는 우리가 일반적으로 Repository를 생성할 때 정의하는 공통적인 메서드들이 들어있다. JpaRepository를 상속하여 인터페이스를 만들면 스프링 데이터 JPA는 프록시 기술을 통해 메서드를 구현한 구현체를 만들어준다.
* 이렇게 만들어진 구현체는 자동으로 스프링 빈으로 등록되기 때문에 @Repository 어노테이션을 붙여줄 필요가 없다.
```java
public interface ItemRepository extends JpaRepository<Item, Long> {
    
    /*
    public Member save(Member member) {
        em.persist(member);
        return member;
    }

    public void delete(Member member) {
        em.remove(member);
    }

    public List<Member> findAll() {
        return em.createQuery("select m from Member m", Member.class)
                .getResultList();
    }

    public Optional<Member> findById(Long id) {
        Member member = em.find(Member.class, id);
        return Optional.ofNullable(member);
    }

    public long count() {
        return em.createQuery("select count(m) from Member m", Long.class)
                .getSingleResult();
    }

    ...

    */
}
```
* JpaRepository<엔티티 타입, 식별자 타입>를 상속한 인터페이스를 만들면 자동으로 주석처리된 것과 같은 메서드를 구현한 구현클래스가 만들어져 스프링 빈으로 등록된다.
## 메서드 생성
```java
public interface ItemRepository extends JpaRepository<Item, Long> {
    
    List<Item> findByItemNameAndStockQuantityGreaterThan(String itemName, int stockQuantity);

    List<Member> findTop3By();
    // List<Member> findTop3ItemNameBy();
    // List<Member> findTop3AAAAAAAAAAAABy();
    

    List<Item> findItemDistinctBy();
}
```
* 기본적으로 find...By, query...By, exists...By, count...By 의 형식으로 함수이름을 만들어야한다.
* ...에는 엔티티 이름이 오든, 엔티티의 필드명이 오든, 아무 상관없는 이름이 오든, 아무 것도 안 적든 JpaRepository\<엔티티 타입, 식별자 타입>에서 정해진 엔티티와 관계된다.
* By뒤에는 where절에 해당한다 보면 된다.
* 간단한 쿼리일 때만 사용하고 복잡한 쿼리의 경우에는 메서드 이름이 너무 길어져서 사용하지 않는 것이 좋다.