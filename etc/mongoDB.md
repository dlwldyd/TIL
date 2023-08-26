# MongoDB
## upsert
```java
public class NameMongoEntity {
    
    private Integer day;

    private List<String> names;

    private Date date;

    private String state;
}
```
```java
// List<NameMongoEntity> : Names
mongoTemplate.bulkOps(BulkOperations.BulkMode.UNORDERED, NameMongoEntity.class).upsert(Names.stream().map(NameMongoEntity -> {
            Query query = new Query();
            query.addCriteria(Criteria.where("day").is(NameMongoEntity.getDay())); // day가 다르면 insert가 실행되고 같으면 update가 실행됨

            // 위의 where 조건이 true 이면 아래의 update문이 실행됨
            // 아래에 없는 필드는 기존 값이 보존되는게 아니라 값이 사라짐 따라서 기존 값을 보존하고 싶어도 전부 update 쿼리에 넣어줘야함
            // 기존에 state 값이 있었더라도 update 시 state 값은 null이 됨
            Update update = new Update();
            update.set("date", NameMongoEntity.getDate());
            update.set("names", NameMongoEntity.getNames());
            return Pair.of(query, update);
        }).collect(Collectors.toList())).execute();
``` 
* BulkOperations.BulkMode.UNORDERED : 각 쿼리가 비동기적으로 실행 됨