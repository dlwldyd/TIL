* -Dfile.encoding=UTF-8 => 파일 사용 시 utf-8 인코딩
* -XX:MinRAMPercentage=50 => 힙 크기 결정, 사용 가능한 전체 메모리 크기가 250MB 미만일 경우 50% 까지만 힙 메모리 사용
* -XX:MaxRAMPercentage=75 => 힙 크기 결정, 사용 가능한 전체 메모리 크기가 250MB 이상일 경우 75% 까지만 힙 메모리 사용
* -XX:+UseG1GC => G1 GC 사용(GC의 한 종류)
* -XX:G1HeapRegionSize=16M => g1 heap region size 결정, region 크기가 크면 객체 크기가 커서 humongous region(하나의 객체가 region을 1개 이상 차지)에 들어갈 수 밖에 없던 객체를 young region에 들어 갈 수 있게 만든다.(https://devblogs.microsoft.com/java/whats-the-deal-with-humongous-objects-in-java, https://blog.leaphop.co.kr/blogs/42)
* -Dsun.net.inetaddr.ttl=0 => DNS 캐시의 TTL 설정, 0이면 캐시를 사용하지 않는다.
* -Dhttp.proxyHost=proxy.example.net => http 사용할 경우 해당 url의 프록시를 통한다.
* -Dhttp.proxyPort=3128 => http 사용할 경우 프록시의 포트
* -Dhttps.proxyHost=proxy.example.net => https 사용할 경우 해당 url의 프록시를 통한다.
* -Dhttps.proxyPort=3128 => https 사용할 경우 프록시의 포트
* -Dhttp.nonProxyHosts=localhost|127.*|192.168.*|10.*|172.16.*|172.17.*|172.18.*|172.19.*|172.20.*|172.21.*|172.22.*|172.23.*|172.24.*|172.25.*|172.26.*|172.27.*|172.28.*|172.29.*|172.30.*|172.31.*|127.0.0.0/8|192.168.0.0/16|10.0.0.0/8|172.16.0.0/12|*.example.io|*.example.com|*.example.net|*.example.com|*.example.com|*.example.in|*.tistory.com|*.melon.com| => 해당 ip, url로 연결할 때는 프록시를 통하지 않고 직접 연결된다.