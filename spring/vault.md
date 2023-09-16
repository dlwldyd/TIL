# Vault
```java
@Configuration
public class VaultConfig extends AbstractVaultConfiguration {

    private final URI vaultUri;

    private final String roleId;

    private final String secretId;

    public VaultConfig(@Value("${vault.url}") URI vaultUri,
                       @Value("${vault.role-id}") String roleId,
                       @Value("${vault.secret-id}") String secretId) {
        this.vaultUri = vaultUri;
        this.roleId = roleId;
        this.secretId = secretId;
    }

    @Override
    public VaultEndpoint vaultEndpoint() {
        return VaultEndpoint.from(vaultUri); // vault 연결
    }

    @Override
    public ClientAuthentication clientAuthentication() {
        AppRoleAuthenticationOptions options = AppRoleAuthenticationOptions.builder() // roleId, secretId로 wallet 연결
                .secretId(AppRoleAuthenticationOptions.SecretId.provided(secretId))
                .roleId(AppRoleAuthenticationOptions.RoleId.provided(roleId))
                .build();

        return new AppRoleAuthentication(options, restOperations());
    }

    // vault에서 값 가져옴
    @Bean 
    public VaultTestProperties VaultTestProperties(VaultOperations operations, ActiveProfileProvider provider) {
        String secret = String.format("secret/test/path", provider.getActiveProfile().getVaultProfileString());
        VaultResponseSupport<VaultTestProperties> response = operations.read(secret, VaultTestProperties.class);
        return response != null ? response.getData() : null;
    }
}
```