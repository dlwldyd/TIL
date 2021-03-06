# 사용자 정의 리포지토리
```java
public interface MemberRepositoryCustom {
    List<Member> findMemberCustom();
}
```
```java
@RequiredArgsConstructor
public class MemberRepositoryImpl implements MemberRepositoryCustom {

    private final EntityManager em;

    @Override
    public List<Member> findMemberCustom() {
        return em.createQuery("select m from Member m", Member.class)
                .getResultList();
    }
}
```
```java
public interface MemberRepository extends JpaRepository<Member, Long>, MemberRepositoryCustom {

    ...

}
```
* JPA를 직접 사용하거나 Querydsl을 사용하는 등 직접 리포지토리 기능을 구현하고 싶을 때 사용한다.
* 내가 구현하고 싶은 메서드를 정의한 인터페이스를 만들고, 해당 인터페이스를 구현한 구현체를 만든다. 그리고 JpaRepository를 상속한 리포지토리 인터페이스에 내가 정의한 인터페이스를 같이 상속시킨다. 그러면 구현체 이름을 규칙에 맞게 정의했다면 내가 만든 구현체가 스프링 빈으로 등록된다.
* 구현체의 이름은 리포지토리 인터페이스 이름 + Impl(MemberRepositoryImpl), 혹은 내가 정의한 인터페이스 이름 + Imple(MemberRepositoryCustomImpl)로 만들어야 스프링 빈으로 등록된다.