# HTTP 헤더
## HTTP 헤더 분류
<center><img src="./../img/HTTP__header.png" width="80%" height="80%"></center>
<center><img src="./../img/http_header2.png" width="80%" height="80%"></center>

* General 헤더 : 메시지 전체에 적용되는 정보
* Request 헤더 : 요청 정보
* Response 헤더 : 응답 정보
* Representation 헤더 : 메시지 바디 정보
## 표현(Representation)
### Content-Type
* 표현 데이터의 형식 설명
* 예)
    + text/html; charset=utf-8
    + application/json
    + image/png
### Content-Encoding
* 표현 데이터를 압축하기 위해 사용
* 데이터를 전달하는 곳에서 압축 후 인코딩 헤더 추가
* 데이터를 읽는 쪽에서 인코딩 헤더의 정보로 압축 해제
* 예)
    + gzip
    + deflate
    + identity
### Content-Language
* 표현 데이터의 자연 언어를 표현
* 예)
  + ko
  + en
  + en-US
### Content-Length
* 표현 데이터의 길이
* 바이트 단위
* Transfer-Encoding(전송 코딩)을 사용하면 Content-Length를 사용하면 안됨
## 협상(Content Negotiation)
* Quality Values(q) 값을 사용하며 0에서 1 사이의 범위이고 클수록 높은 우선순위를 가짐, 생략하면 q=1
  + 예) Accept-Language: ko-KR,ko;q=0.9,en-US;q=0.8,en;q=0.7
    1. ko-KR;q=1 (q생략)
    2. ko;q=0.9
    3. en-US;q=0.8
    4. en:q=0.7
* 구체적인 것이 우선한다.
  + 예) Accept: text/\*, text/plain, text/plain;format=flowed, \*/\*
    1. text/plain;format=flowed
    2. text/plain
    3. text/\*
    4. \*/\*
* 구체적인 것을 기준으로 미디어 타입을 맞춘다.
  + Accept: text/\*;q=0.3, text/html;q=0.7, text/html;level=1, text/html;level=2;q=0.4, \*/\*;q=0.5
    - text/html;level=1 -> q=1
    - text/html -> q=0.7
    - text/plain -> q=0.3
    - image/jpeg -> q=0.5
    - text/html;level=2 -> q=0.4
    - text/html;level=3 -> q=0.7
## 전송 방식
* 단순 전송
    + content-length에 보낼 데이터의 길이를 명시한다.
    + 한번에 보내고 한번에 받는다.
* 압축 전송
    + content-length 뿐만 아니라 content-encoding 헤더 또한 명시한다.
    + 한번에 보내고 한번에 받는다.
* 분할 전송
    + content-length를 명시하지 않는다.
    + transfer-encoding: chunked로 명시하여 사용한다.
* 범위 전송
    + content-range를 명시하여 특정 범위만 요청할 수 있다.
    + 응답은 범위에 해당하는 데이터만 응답한다.
## 일반 정보
* From : 유저 에이전트의 이메일 정보
* Referer : 현재 요청된 페이지의 이전 웹 페이지 주소. Referer를 사용해서 유입 경로 분석이 가능하다.
* User-Agent : 클리이언트의 애플리케이션 정보(웹 브라우저 정보, 등등). 어떤 종류의 브라우저에서 장애가 발생하는지 파악이 가능하다.
* Server : 요청을 처리하는 ORIGIN 서버의 소프트웨어 정보
* Date : 메시지가 발생한 날짜와 시간. 응답에서 사용한다.
* Host : 요청한 호스트 정보(도메인). 하나의 IP 주소에 여러 도메인이 적용되어 있을 때 유용하다.
## 특별한 정보
* Location : 웹 브라우저는 3xx 응답의 결과에 Location 헤더가 있으면, Location 위치로 자동 이동한다.(리다이렉트)
* Allow : 405 (Method Not Allowed) 에서 응답에 포함해야한다.
* Retry-After : 유저 에이전트가 다음 요청을 하기까지 기다려야 하는 시간. 503 (Service Unavailable)응답에 포함 되어야 한다.
## 인증
* Authorization : 클라이언트 인증 정보를 서버에 전달
* WWW-Authenticate : 리소스 접근시 필요한 인증 방법 정의, 401 Unauthorized 응답과 함께 사용
## 쿠키
* Set-Cookie : 서버에서 클라이언트로 쿠키 전달(응답)
* Cookie : 클라이언트가 서버에서 받은 쿠키를 저장하고, HTTP 요청시 서버로 전달