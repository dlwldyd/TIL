# HTTP 상태 코드
클라이언트가 보낸 요청의 처리상태를 응답에서 알려주는 기능이다.
* 1xx (Informational): 요청이 수신되어 처리 중
* 2xx (Successful): 요청 정상 처리됨
* 3xx (Redirection): 요청을 완료하려면 추가 행동이 필요
* 4xx (Client Error): 클라이언트 오류, 잘못된 문법등으로 서버가 요청을 수행할 수 없음
* 5xx (Server Error): 서버 오류, 서버가 정상 요청을 처리하지 못함
## 1xx (Informational)
* 거의 사용되지 않음
## 2xx (Successful)
* 200 OK : 요청 성공
* 201 Created : 요청에 성공해서 새로운 리소스가 생성됨
* 202 Accepted : 요청이 접수되었으나 처리가 완료되지 않음
    + 배치 처리 같은 곳에서 사용됨
* 204 No Content : 서버가 요청을 성공적으로 수행했지만, 응답 페이로드 본문에 보낼 데이터가 없음
    + 웹 문서의 save 버튼 같은 곳에서 사용됨
## 3xx (Redirection)
웹 브라우저는 3xx 응답의 결과에 Location 헤더가 있으면, Location 위치로 자동 이동(Redirect)
### 영구적인 Redirection
리소스의 URI가 영구적으로 이동했음을 나타냄
* 301 Moved Permanently : 리다이렉트시 요청 메서드가 GET으로 변하고, 본문이 제거될 수 있음
* 308 Permanent Redirect : 301과 기능은 같지만 차이점은 리다이렉트시 요청 메서드와 본문 유지(처음 POST를 보내면 리다이렉트도 POST 유지)
### 일시적인 Redirection
리소스의 URI가 일시적으로 이동했음을 나타냄
* 302 Found : 리다이렉트시 요청 메서드가 GET으로 변하고, 본문이 제거될 수 있음
* 307 Temporary Redirect : 302와 기능은 같지만 차이점은리다이렉트시 요청 메서드와 본문 유지(처음 POST를 보내면 리다이렉트도 POST 유지)
* 303 See Other : 302와 기능은 같지만 차이점은 리다이렉트시 요청 메서드가 무조건 GET으로 변경됨
### 기타 Redirection
* 304 Not Modified
    + 캐시를 목적으로 사용
    + 클라이언트에게 리소스가 수정되지 않았음을 알려준다. 따라서 클라이언트는 로컬PC에 저장된 캐시를 재사용한다. (캐시로 리다이렉트 한다.)
    + 304 응답은 로컬 캐시를 사용해야 하므로 응답에 메시지 바디를 포함하면 안된다.
## 4xx (Client Error)
* 400 Bad Request : 요청 구문, 메시지 등 오류
* 401 Unauthorized : 인증 되지 않음, 401 오류 발생시 응답에 WWW-Authenticate 헤더와 함께 인증 방법을 설명
* 403 Forbidden : 인증 자격 증명은 있지만, 접근 권한이 불충분한 경우
* 404 Not Found : 요청 리소스가 서버에 없음, 또는 클라이언트가 권한이 부족한 리소스에 접근할 때 해당 리소스를 숨기고 싶을 때에도 사용
## 5xx (Server Error)
* 500 Internal Server Error : 서버 내부 문제로 오류 발생
* 503 Service Unavailable : 서버가 일시적인 과부하 또는 예정된 작업으로 잠시 요청을 처리할 수 없음, Retry-After 헤더 필드로 얼마뒤에 복구되는지 보낼 수도 있다.