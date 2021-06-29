# HTTP 요청 구조

HTTP 요청 메시지는 크게 세 부분으로 구성되어 있다.

1. Start Line
2. Headers
3. Body

## Start Line

이름 그대로 HTTP 요청의 시작줄이다. "ping" 엔드포인트에 GET HTTP 요청을 보낸다면 해당 HTTP 요청의 start line은
`GET /ping HTTP/1.1`
이다.

start line은 세 부분으로 구성되어 있다.

- HTTP 메소드
- Request target
- HTTP Version

### HTTP 메소드

HTTP 메소드는 해당 HTTP 요청이 의도하는 액션을 정의하는 부분이다. HTTP 메소드에는 GET, POST, PUT, DELETE, OPTIONS 등 여러 메소드들이 있다.

### Request target

Request target은 해당 HTTP 요청이 전송되는 목표 주소를 말한다. 위의 예로 본다면 request target은 "/ping"이 된다.

### HTTP version

HTTP version은 이름 그대로 해당 요청의 HTTP 버전을 나타낸다. 버전에는 현재 "1.0", "1.1" 그리고 "2.0"이 있다.
HTTP 버전을 명시하는 이유는 HTTP 버전에 따라 HTTP 요청 메시지의 구조나 데이터가 약간 다를 수 있으므로 서버가 받은 요청의 HTTP version에 맞추어서 응답을 보낼 수 있도록 하기 위함이다.

## 헤더

start line 다음에 나오는 부분은 헤더(header)다. 헤더 정보는 HTTP 요청 그 자체에 대한 정보를 담고 있다. 예를 들어, HTTP 요청 메시지의 전체 크기(Content-Length) 같은 정보를 담고 있다.  
헤더는 파이썬의 딕셔너리 처럼 키와 밸류로 되어 있다.
HTTP 요청의 Host 헤더의 경우는
`HOST : google.com`
key는 HOST 이고 value는 google.com 이다.

**자주 사용되는 헤더 정보**

- Host
  - 요청이 전송되는 target의 호스트의 URL주소를 알려 주는 헤더
- User-Agent
  - 요청을 보내는 클라이언트의 대한 정보(웹 브라우저에 대한 정보)
  - 예:Mozilla/5.0 (Macintosh; Intel Mac OS X 10_15_7) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/91.0.4472.114 Safari/537.36
- Accept
  - 해당 요청이 받을 수 있는 응답 body 데이터 타입을 알려 주는 헤더
  - MIME type이 value로 지정된다. JSON 데이터 타입을 요청하는 경우에는 application/json이고 모든 데이터 타입을 다 허용하는 경우에는 `*/*`로 지정해 주면 된다.
- Connection
  - 해당 요청이 끝난 후에 클라이언트와 서버가 계속해서 네트워크 연결을 유지할 것인지 아니면 끊을 것인지에 대해 알려주는 헤더
  - connection 헤더의 값이 keep-alive이면 앞으로도 계속해서 HTTP 요청을 보낼 에정이므로 네트워크 연결을 유지하라는 뜻이다.
  - connection 헤더의 값이 close라고 지정되면 더 이상 HTTP 요청을 보내지 않을 것이므로 네트워크 연결을 닫아도 된다는 뜻이다.
- Content-Type
  - HTTP 요청이 보내는 메시지 body의 타입을 알려 주는 헤더다
  - Accept와 마찬가지로 MIME type이 사용된다.
- Content-Length
  - HTTP 요청이 보내는 메시지 body의 총 사이즈를 알려 주는 헤더다

## Body

HTTP 요청 메시지에서 body 부분은 HTTP 요청이 전송하는 데이터를 담고 있는 부분이다. 전송하는 데이터가 없다면 body 부분은 비어 있다.
