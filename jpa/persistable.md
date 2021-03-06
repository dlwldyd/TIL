# Persistable<> 인터페이스
## JpaRepository 구현체 구조(save 메서드를 예로 들어서)
```java
@Repository
@Transactional(readOnly = true)
public class SimpleJpaRepository<T, ID> implements JpaRepositoryImplementation<T, ID> {

    ...

    @Transactional
    @Override
    public <S extends T> S save(S entity) {

        Assert.notNull(entity, "Entity must not be null.");

        if (entityInformation.isNew(entity)) {
            em.persist(entity);
            return entity;
        } else {
            return em.merge(entity);
        }
    }

    ...
}
```
* save메서드를 보면 엔티티가 새로 생성된 엔티티이면 persist()를 하고 그렇지 않으면 merge()를 한다.
* 새로 생성된 엔티티인지 아닌지를 판단할 때는 엔티티의 식별자가 @GeneratedValue 어노테이션을 통해 자동 생성된다고 가정하고 해당 엔티티의 식별자를 보고 판단하는데, 만약 식별자가 primitive 타입일 때는 식별자가 0이면 새로 생성된 엔티티이고 0이 아니면 새로 생성된 엔티티가 아니라 판단한다. 그리고 primitive 타입이 아닌 경우에는 식별자가 null이면 새로 생성된 엔티티이고 null이 아니면 새로 생성된 엔티티가 아니라 판단한다. 하지만 만약 엔티티의 식별자가 @GeneratedValue 어노테이션을 통해 자동 생성되는 식별자가 아니라면 이러한 메커니즘으로 엔티티가 새로생성된 엔티티인지 판단할 수 없다.
## Persistable<> 구현
```java
@Entity
@Getter
@EntityListeners(AuditingEntityListener.class)
@NoArgsConstructor(access = AccessLevel.PROTECTED)
public class Item implements Persistable<String> {

    @Id
    private String id;

    @CreatedDate
    private LocalDateTime createdDate;

    public Item(String id) {
        this.id = id;
    }

    @Override
    public boolean isNew() {
        return createdDate == null;
    }
}
```
* persist()를 호출해도 되는데 merge()를 호출하는 문제를 해결하기 위해 이러한 문제를 가진 엔티티에 Persistable<> 인터페이스를 구현하고 isNew()메서드를 오버라이드하여 해당 엔티티가 새로 생성된 엔티티인지 판단하는 로직을 구현한다.
* 일반적으로 모든 엔티티는 @CreatedDate, @LastModifiedDate가 적용된 필드를 가지고 있기 때문에 이러한 필드를 이용하여 새로 생성된 엔티티인지 판단한다.
* Persistable<>에 들어가는 제네릭은 식별자의 타입이다.