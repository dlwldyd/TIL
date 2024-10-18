# BigQuery
## Gcp 연결
```yml
spring:
  cloud:
    gcp:
      project-id: projectId
      credentials:
        location: /usr/local/key.json
      bigquery:
        project-id: projectId
        dataset-name: dataset
```
```java
@Repository
@RequiredArgsConstructor
public class BigQueryRepository {

    private final BigQuery bigQuery;

    @Override
    public List<RouteChangeData> getRouteChange(String day, String reqId, Integer chReason) {
        String query = "select * from `dataset_name.table_name`";
        QueryJobConfiguration queryConfig = QueryJobConfiguration.newBuilder(query).build();
        List<BigQueryEntity> result = new ArrayList<>();
        try {
            for (FieldValueList row : bigQuery.query(queryConfig).iterateAll()) {
                BigQueryEntity entity = BigQueryEntity.builder()
                        .field1(row.get("field1").getStringValue())
                        .day(row.get("day").getStringValue()) // bigquery에서 타입이 DATE면 string으로 받음
                        .field2(row.get("field2").getLongValue())
                        .build();
                result.add(entity);
            }
            return result;
        } catch (InterruptedException e) {
            throw new RuntimeException(e);
        }
    }
}
```
```kotlin
val job = bigQuery.create(
    JobInfo.newBuilder(queryJobConfiguration)
        .setJobId(
            JobId.of(
                "my_job_%s_%s".format(
                    UUID.randomUUID().toString(),
                    DateFormatUtils.format(System.currentTimeMillis(), "HH_mm_ss_SSS")
                )
            )
        )
        .build()
)
job.status.error?.let { error ->
    throw BigQueryRuntimeException(error.reason, error.message)
}
```
* yml 파일에 projectId, credentials(필요한 경우) 등을 입력하면 자동으로 BigQuery 객체를 만들어서 bean으로 등록함
* 조회할 때는 BigQuery를 사용하고 데이터를 넣을 때는 BigQueryTemplate을 사용하는듯?
* job id를 직접 넣어줄 때 job id가 겹치면 job = null 이 되고(job id 가 이미 있으면 job이 생성 안됨) 해당 job을 가지고 query를 실행하면 NullPointerException 발생
## 파일 형태가 아니라 외부에서 credential 값 가지고 오기
```java
public class DefaultCredentialsProvider implements CredentialsProvider {

  ...

  public DefaultCredentialsProvider(CredentialsSupplier credentialsSupplier) throws IOException {
    List<String> scopes = resolveScopes(credentialsSupplier.getCredentials().getScopes());
    Resource providedLocation = credentialsSupplier.getCredentials().getLocation();
    String encodedKey = credentialsSupplier.getCredentials().getEncodedKey();

    if (providedLocation != null) {
      this.wrappedCredentialsProvider =
          FixedCredentialsProvider.create(
              GoogleCredentials.fromStream(providedLocation.getInputStream()).createScoped(scopes)); // 기본적으로 사용하는 DefaultCredentialsProvider에서는 yml파일에 입력된 service account key 파일의 경로로부터 credential을 읽어온다.
    } else if (StringUtils.hasText(encodedKey)) {

        ...

    } else {

      ...

    }

    ...

  }
}
```
```java
@Component
public class GcpCredentialProvider implements CredentialsProvider {
    private static final String DEFAULT_SCOPES_PLACEHOLDER = "DEFAULT_SCOPES";
    private static final List<String> CREDENTIALS_SCOPES_LIST =
            Collections.unmodifiableList(
                    Arrays.stream(GcpScope.values()).map(GcpScope::getUrl).collect(Collectors.toList()));

    private final CredentialsProvider wrappedCredentialsProvider;

    @Override
    public Credentials getCredentials() throws IOException {
        return this.wrappedCredentialsProvider.getCredentials();
    }

    public GcpCredentialProvider(CredentialsSupplier credentialsSupplier,
                                 VaultGcpKeyProperties vaultGcpKeyProperties) throws IOException {
        List<String> scopes = resolveScopes(credentialsSupplier.getCredentials().getScopes());

        this.wrappedCredentialsProvider =
                FixedCredentialsProvider.create(
                        GoogleCredentials.fromStream(new ByteArrayInputStream(vaultGcpKeyProperties.getKey().getBytes())).createScoped(scopes)); // yml에 입력된 파일의 경로로부터 credential을 읽어 오는게 아니라 vault로 읽어 오도록 수정
    }
}
```
* 기존에는 DefaultCredentialProvider가 yml에 입력된 파일 경로로부터 credential을 읽어오도록 되어있다. 이를 내가 원하는 곳으로 부터 읽어 오도록 수정하여 bean으로 등록한다.
## config 변경
```yml
spring:
  autoconfigure:
    exclude:
      - com.google.cloud.spring.autoconfigure.core.GcpContextAutoConfiguration
      - com.google.cloud.spring.autoconfigure.bigquery.GcpBigQueryAutoConfiguration
```
```java
@Bean
public CredentialsProvider googleCredentials() throws IOException {
    return new GcpCredentialProvider(this.gcpProperties, vaultGcpKeyProperties);
}
```
```java
@Bean
public BigQuery bigQuery(GcpProjectIdProvider projectIdProvider) throws IOException {
    BigQueryOptions.Builder builder = BigQueryOptions.newBuilder()
            .setProjectId(projectIdProvider.getProjectId())
            .setHeaderProvider(new UserAgentHeaderProvider(GcpBigQueryConfig.class))
            .setRetrySettings(RetrySettings.newBuilder()
                    .setMaxAttempts(5)
                    .setRetryDelayMultiplier(1.5)
                    .setTotalTimeout(Duration.ofMinutes(5))
                    .build())
            .setTransportOptions(HttpTransportOptions.newBuilder()
                    .setConnectTimeout(60000)
                    .setReadTimeout(20000)
                    .build());

    BigQueryOptions bigQueryOptions = builder
            .setCredentials(this.credentialsProvider.getCredentials())
            .build();
    return bigQueryOptions.getService();
}
```
- PostBeanProcessor 가 안먹힐 때 위 방법 사용
- GcpContextAutoConfiguration, GcpBigQueryAutoConfiguration 의 auto configuration 을 exclude 하고 해당 클래스에서 등록하는 빈 대신 내가 원하는대로 커스텀 해서 등록