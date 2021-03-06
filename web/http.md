# HTTP 기초
## HTTP 특징
* 클라이언트 서버 구조
* 무상태 프로토콜, 비연결성
* 단순함, 확장가능
### 클라이언트 서버 구조
<center><img src="./../img/client_server.png" width="60%" height="60%"></center>

* 클라이언트는 서버에 요청을 보내고 응답을 대기
* 서버가 요청에 대한 결과를 만들어서 응답
### Stateless와 Stateful의 차이
#### Stateful Architecture
<center><img src="./../img/stateless_architecture.png" width="60%" height="60%"></center>

* Client A에 대한 정보는 Server #1에 저장되어 있고 Server #2에는 저장되어있지 않다.
* 따라서 Client A가 Server #2에 연결하기 위해서는 세션 인증을 처음부터 다시 해야할 필요가 있고, 이러한 불필요한 작업을 없애기 위해 Client A의 요청은 항상 Server #1을 향하도록 관리할 필요가 있다.
#### Stateless Architecture
<center><img src="./../img/stateless_architecture.png" width="60%" height="60%"></center>

* Server #1과 Server #2는 DB를 통해 Client A에 대한 정보에 접근 가능하다.
* 따라서 Stateless 구조에서는 Client A의 요청을 특정서버를 향하도록 관리할 필요가 없다.
#### Stateless를 사용하는 이유
<center><img src="./../img/stateful_scaleout.png" width="60%" height="60%"></center>

Stateful 서비스의 경우 트래픽의 급증으로 서버를 확장할 때 client의 세션 정보가 확장된 서버에 저장되어 있지 않다. 따라서 Stateful 서비스를 확장하려면 확장된 서버에 세션 정보를 옮기는 등 부수적인 작업이 필요하다.
<center><img src="./../img/stateless_scaleout.png" width="60%" height="60%"></center>

반면에 Stateless 서비스의 경우에는 Client의 세션 정보가 서버에 저장되어 있지 않으므로 부수적인 작업을 할 필요가 없다.
### TCP연결을 유지하지 않는 이유(비연결성)
#### 연결을 유지할 경우
<center><img src="./../img/connect.jpeg" width="60%" height="60%"></center>

TCP 연결을 유지한다면 서버에 트래픽이 몰릴 때 서버의 TCP backlog queue가 가득차서 다른 클라이언트의 요청을 처리할 수 없게 된다.
#### 연결을 유지하지 않을 경우
<center><img src="./../img/connect.jpeg" width="60%" height="60%"></center>

반면에 TCP 연결을 유지하지 않을 경우 서버에 트래픽이 몰리더라도 HTTP는 일반적으로 초단위 이하로 빠른 응답을 하기 때문에 수많은 요청이 있더라도 실제로 서버가 동시에 처리하는 요청은 매우 적다. 따라서 서버는 서버의 자원을 효율적으로 사용할 수 있다.
### HTTP 메시지 구조
<center><img src="./../img/http_message.png" width="60%" height="60%"></center>

#### start line(request line)
* GET, POST 등 서버가 해야할 동작을 지정하는 HTTP 메서드
* 요청 대상이 지정되어 있는 도메인과 경로
* HTTP 버전 정보
* HTTP 상태 코드
#### HTTP 헤더
* HTTP 전송에 필요한 모든 부가정보
* 필요시 임의의 헤더 추가 가능
#### 공백 라인
* HTTP 헤더와 HTTP 메시지 바디를 구분
#### HTTP 메시지 바디
* 실제 전송할 데이터
* HTML 문서, 이미지, 영상, JSON 등 byte로 표현할 수 있는 모든 데이터 전송 가능