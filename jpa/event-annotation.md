# 이벤트 어노테이션
```java
@MappedSuperclass
@Getter
@Setter
public class JpaBaseEntity {

    @Column(updatable = false)
    private LocalDateTime createdDate;

    private LocalDateTime updatedDate;

    // @PostPersist persist 후에 실행됨
    @PrePersist //persist 전에 실행됨
    public void prePersist() {
        LocalDateTime now = LocalDateTime.now();
        createdDate = now;
        updatedDate = now;
    }

    // @PostUpdate update 후에 실행됨
    @PreUpdate //update 전에 실행됨
    public void preUpdate() {
        updatedDate = LocalDateTime.now();
    }
}
```
* JPA에는 @PrePersist, @PostPersist, @PreUpdate, @PostUpdate라는 주요 이벤트 어노테이션이 있다.
* @PrePersist, @PostPersist는 persist 메서드가 호출되는 시점 전후에 해당 어노테이션이 적용된 메서드가 실행된다.
* @PreUpdate, PostUpdate는 update쿼리가 날라간 시점 전후(flush() 전후)에 해당 어노테이션이 적용된 메서드가 실행된다.