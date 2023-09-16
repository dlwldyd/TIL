# JsonDeserializer, JsonPojoBuilder
```java
@Getter
@Builder
@JsonDeserialize(builder = PosData.PosDataBuilder.class) // builder 넣기
public class PosData {

    private Float x;
    private Float y;
    private Float z;
    @Builder.Default
    private Long time = 1L;

    public PosData(Float x, Float y, Float z, Long time) {
        this.x = x;
        this.y = y;
        this.z = z;
        this.time = time;
    }

    @JsonPOJOBuilder(withPrefix = "") // withPrefix 필수
    public static class PosDataBuilder {
    }
}
```
* ObjectMapper가 json을 deserialize할 때 NoArgConstructor가 필요하다 하지만 NoArgConstructor를 추가하면 사용하지도 않는 생성자가 추가되는건데, 이렇게 필요없는 생성자를 추가하는 대신에 builder를 통해 json을 deserialize 하도록 하는게 JsonDeserializer이다.
* JsonPojoBuilder는 JsonDeserializer와 세트로 따라다니는 것
* @Builder.Default 어노테이션을 사용해 default 값 설정 가능