# Auditing
```java
@EntityListeners(AuditingEntityListener.class) // 필수
@MappedSuperclass
@Getter
@Setter
public class BaseEntity {

    @CreatedDate
    @Column(updatable = false)
    private LocalDateTime createdDate;

    @LastModifiedDate
    private LocalDateTime lastModifiedDate;

    @CreatedBy
    @Column(updatable = false)
    private String createdBy;

    @LastModifiedBy
    private String lastModifiedBy;
}
```
```java
@EnableJpaAuditing //필수
@Configuration
public class AppConfig {

    @Bean
    public AuditorAware<String> auditorProvider() {
        return () -> Optional.of(UUID.randomUUID().toString());
    }
```
* @CreatedDate : 해당 어노테이션이 적용된 필드에 데이터의 생성 일자가 들어간다.
* @LastModifiedDate : 해당 어노테이션이 적용된 필드에 마지막으로 데이터가 업데이트된 일자가 들어간다.
* @CreatedBy : 해당 어노테이션이 적용된 필드에 데이터를 누가 생성했는지가 들어간다.
* @LastModifiedBy : 해당 어노테이션이 적용된 필드에 마지막으로 데이터를 누가 업데이트했는지가 들어간다.
* 위의 어노테이션이 들어간 클래스에는 @EntityListeners(AuditingEntityListener.class)가 필수적으로 들어가야한다.
* @CreatedBy, @LastModifiedBy를 쓴다면 설정 클래스에 @EnableJpaAuditing를 적용하고 AuditorAware<>클래스를 스프링 빈으로 등록하여 @CreatedBy, @LastModifiedBy가 적용된 필드에 어떤 정보가 들어가야 하는지를 정의해야 한다.
* 일반적으로는 @CreatedDate, @LastModifiedDate와 @CreatedBy, @LastModifiedBy를 분리하여 적용한다.