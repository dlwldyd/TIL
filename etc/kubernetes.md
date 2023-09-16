# 쿠버네티스 설정
## deployment
```yml
apiVersion: apps/v1
kind: Deployment
metadata:
  labels:
    app: app-name
  name: app-name-deploy
  namespace: default # 어느 namespace에 들어갈지
spec:
  replicas: 3 # replica 개수
  minReadySeconds: 0
  selector:
    matchLabels:
      app: loc-test-platform
  strategy:
    rollingUpdate:
      maxSurge: 25%
      maxUnavailable: 1
    type: RollingUpdate # 배포할 때 RollingUpdate(몇개 배포하면서 나머지는 서버돌리면서 무중단으로)
  template:
    metadata:
      labels:
        app: loc-test-platform
    spec:
      dnsConfig:
        nameservers:
          - 10.20.30.40
        options:
          - name: timeout
            value: "1"
          - name: attempts
            value: "2"
      dnsPolicy: None
      terminationGracePeriodSeconds: 20
      containers:
        - name: loc-test-platform
          image: idock.daumkakao.io/mobility-location/loc-test-platform-dev:latest
          env:
            - name: JAVA_TOOL_OPTIONS # 빌드할 때 옵션 줄 수 있음
              value: >
                -Dfile.encoding=UTF-8
                -Xms6G
                -Xmx6G
                -XX:+UseG1GC
                -XX:G1HeapRegionSize=16M
                -Dsun.net.inetaddr.ttl=0
            - name: APP_PROFILES_ACTIVE # 시스템 환경변수 설정 가능
              value: dev
            - name: SPRING_PROFILES_ACTIVE
              value: dev

          imagePullPolicy: "Always"
          ports:
            - containerPort: 8080
          resources:
            limits:
              cpu: 1500m
              memory: 3000Mi
            requests:
              cpu: 1500m
              memory: 3000Mi
          lifecycle:
            preStop:
              exec:
                command: [ "/bin/sleep","10" ]
```
## ingress
```yml
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: app-name-ingress
  namespace: default # 어느 namespace에 속할지
  annotations:
    nginx.ingress.kubernetes.io/rewrite-target: /
    nginx.ingress.kubernetes.io/ssl-redirect: "false"
spec:
  rules:
    - host: http://my-url.com # 부여받은 url
      http:
        paths:
          - path: / # prefix가 '/' 면 해당 ingress로 라우팅됨
            pathType: Prefix
            backend:
              service:
                name: app-name-svc
                port:
                  number: 8080
```
## service
```yml
apiVersion: v1
kind: Service
metadata:
  name: app-name-svc
spec:
  ports:
    - port: 80
      targetPort: 8080
      protocol: TCP
  selector:
    app: app-name

```
## ingress-vip(load balancer)
```yml
kind: Service
apiVersion: v1
metadata:
  name: ingress-vip
  namespace: ingress-nginx
spec:
  externalTrafficPolicy: Local
  selector:
    app: app-name
  type: LoadBalancer
  ports:
    - name: web-port
      port: 80
      targetPort: 80
```