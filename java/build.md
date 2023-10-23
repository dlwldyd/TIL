* -Dfile.encoding=UTF-8 => 파일 사용 시 utf-8 인코딩
* -XX:MinRAMPercentage=50 => 힙 크기 결정, 사용 가능한 전체 메모리 크기가 250MB 미만일 경우 50% 까지만 힙 메모리 사용
* -XX:MaxRAMPercentage=75 =>  => 힙 크기 결정, 사용 가능한 전체 메모리 크기가 250MB 이상일 경우 75% 까지만 힙 메모리 사용
* -XX:+UseG1GC => G1 GC 사용(GC의 한 종류)
* -XX:G1HeapRegionSize=16M => g1 heap region size 결정, region 크기가 크면 객체 크기가 커서 humongous region(하나의 객체가 region을 1개 이상 차지)에 들어갈 수 밖에 없던 객체를 young region에 들어 갈 수 있게 만든다.(https://devblogs.microsoft.com/java/whats-the-deal-with-humongous-objects-in-java, https://blog.leaphop.co.kr/blogs/42)
* -Dsun.net.inetaddr.ttl=0 => DNS 캐시의 TTL 설정, 0이면 캐시를 사용하지 않는다.
