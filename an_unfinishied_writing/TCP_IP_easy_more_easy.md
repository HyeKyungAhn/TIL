# TCP/IP 쉽게, 더 쉽게

## 네트워크 서비스와 애플리케이션 계층
- 개관
    - 애플리케이션 계층은 **서비스 제공**
    - 이 계층의 프로토콜은 서비스 제공을 위해 서버와 클라이언트 간 다양한 메시지나 데이터를 주고 받는 일을 하게 함
    - 서비스 구현 시, 새로운 프로토콜이 아닌 웹 페이지를 다룰 때 사용하는 **HTTP 프로토콜을 활용**하는 서비스를 (좁은 의미의)***웹 서비스***라고 한다. 더 짧은 시간 내 적은 노력으로 개발 및 개선을 할 수 있다.

### 애플리케이션 계층
- 역할 : 사용자가 **직접 사용하며 체감하는** 서비스 제공
- 범주 : 데이터 전송 관련 계층을 제외한 모든 영역
- 영향 : 서비스의 종류, 동작 방식을 결정
- 정의 대상 : 애플리케이션 프로그램 간의 통신을 정의
    - 다른 계층은 컴퓨터간 통신을 정의
- 서비스 별로 자신만의 프로토콜을 만들어 사용

### 사용자가 직접 사용하는 애플리케이션 계층 프로토콜
HTTP, POP, SMTP, IMAP, SMB, AFP, FTP, Telnet, SSH

### 사용자가 간접적으로 사용하는 애플리케이션 계층 프로토콜
- OS나 다른 애플리케이션 프로토콜이 간접적으로 사용
- 주로 **인터넷이나 LAN의 원활한 사용**을 위해 사용

|프로토콜|동작 방식|
|---|---|
|DNS(Domain Name System)|도메인명과 IP 어드레스의 정보를 서로 변환할 때 사용|
|DHCP(Dynamic Host Configuration Protocol)|LAN 내의 컴퓨터에게 IP 어드레스를 할당할 때 사용한다.|
|SSL/TLS(Secure Sockets Layer/Transport Layer Security)|통신 데이터를 암호화, 주요 정보를 안전하게 주고 받기위해 사용|
|NTP(Network Time Protocol)|네트워크에 연결된 장비들의 시스템 시간을 동기화할 떄 사용|
|LDAP(Lightwight Directory Access Protocol)|네트워크에 연결된 자원(사용자, 장비)의 통합 관리에 필요한 디렉터리 서비스를 제공할 때 사용|

## 웹페이지를 전송하는 HTTP
### 웹페이지가 표시되기까지의 과정
- 웹 브라우저가 웹서버로 웹페이지 요청시, **HTML 파일 형식**으로 응답한다.
- 응답 파일에는 HTML 파일 형식 외에, 웹페이지를 표시하기 위해 필요한 파일(CSS, JS, JPEG..)의 정보도 포함되어있다
- HTML 파일을 렌더링할 때 필요한 파일의 정보를 읽으면 새로운 요청을 보내어 그 파일을 응답받음
### HTTP 메시지
- HTTP 프로토콜을 사용하여 주고 받는 데이터가 HTTP 메시지
- 요청과 응답 두 가지 형태가 있다.
- HTTP의 가장 큰 특징은 무상태성(stateless)이다. 상태 정보를 저장하지 않는 통신 형식을 말한다.
### HTTP 요청과 URL
- HTTP 요청을 할 때, URL(Uniform R Locator) 문자열을 사용
    - URL 형식
    http://www.domain.com/directory/fileName
    스키마(http) - 호스트(www) - 도메인(domain.com) - 디렉토리이름(directory) - 파일이름(filename)
- HTTP 요청 메시지 형식

    |GET /directory/filename HTTP/1.1|요청 정보 라인|
    |---|---|
    |Host: www.domain.com|       헤더|
    |브라우저 관련 정보들|          헤더|
    - 요청 정보라인에는 요청방식, 디렉터리/파일 이름, 프로토콜 버전정보가 포함된다.
    - POST 방식의 경우에는 헤더 뒤에 요청내용인 메시지 바디가 있음
    - 어느 서버의 디렉토리의 파일을 찾아달라는 것

### HTTP 응답과 상태코드
- 응답 상태를 나타내기 위해 상태코드가 있다
    - 100번대는 정보
    - 200번대는 성공
    - 300번대는 **경로 전환**
    - 400번대는 에러를 의미
- HTTP 응답 메시지 형식

    |HTTP/1.1 200 OK|응답 정보 라인|
    |---|---|
    |파일 관련 내용|       헤더|
    |파일 갱신 날짜 정보|          헤더|
    |파일 content type|          헤더|
    |파일 크기 등등|          헤더|
    |(빈줄)||
    |HTTP 파일 문자열|          바디|