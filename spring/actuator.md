# spring actuator
```yml
management:
  health:
    livenessstate:
      enabled: true
    readinessstate:
      enabled: true
  endpoint:
    health:
      probes:
        enabled: true
  server:
    port: 9000 # 포트 바꾸고 싶을 때만
```
- `localhost:9000/actuator/health` 로 readiness, liveness health check 가능
- management.endpoint.health.probes.enabled = true 넣으면 `localhost:9000/actuator/health/readiness`, `localhost:9000/actuator/health/liveness` 로도 가능