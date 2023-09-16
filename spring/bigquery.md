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
* yml 파일에 projectId, credentials(필요한 경우) 등을 입력하면 자동으로 BigQuery 객체를 만들어서 bean으로 등록함
* 조회할 때는 BigQuery를 사용하고 데이터를 넣을 때는 BigQueryTemplate을 사용하는듯?
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