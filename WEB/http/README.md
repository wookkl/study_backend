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

# HTTP 응답 구조

HTTP 응답 메시지의 구조도 요청 메시지와 마찬가지로 크게 세 부분으로 구성되어 있다.

1. Status Line
2. Headers
3. Body

## Status Line

HTTP 응답 메시지의 상태를 간략하게 요약하여 알려주는 부분이다.
세 부분으로 구성되어 있다.
`HTTP/1.1 404 Not Found`

1. HTTP Version: HTTP 버전을 나타낸다.
2. Status Code: HTTP 응답 상태를 미리 지정되어 있는 숫자로 된 코드로 나타내준다.
3. Status Text: HTTP 응답 상태를 간략하개 글로 설명해주는 부분이다.

## 헤더

HTTP 응답의 헤더 부분은 HTTP 요청의 헤더 부분과 동일하다. 다만 HTTP 응답에서만 사용되는 헤더 값들이 있다.

## Body

HTTP 응답 메시지의 body도 HTTP 요청 메시지의 body와 동일하다.

# 자주 사용되는 HTTP 메소드

## GET

POST 메소드와 함께 가장 자주 사용되는 HTTP 메소드이다. 어떠한 데이터를 서버로부터 요청할 때 주로 사용하는 메소드이다.

## POST

GET 메소드와 함께 가장 자주 사용되는 메소드다. 데이터를 생성하거나 수정 및 삭제 요청을 할 때 주로 사용되는 메소드다.

## OPTIONS

OPTIONS 메소드는 주로 특정 엔드포인트에서 허용되는 메소드들이 무엇이 있는 지 알고자 하는 요청에서 사용되는 HTTP 메소드다. 엔드포인트는 허용하는 HTTP 메소드가 지정되도록 되어 있으며, 허용하지 않는 HTTP 메소드의 요쳥이 들어오면 405 Method Not Allowed 응답을 보내게 된다.

엔드포인트가 어떠한 HTRP 메소드 요청을 허용하는지 알고자 할 때 OPTIONS 요청을 보내게 된다.

```
HTTP/1.0 200 OK
Allow: GET, HEAD, OPTIONS
Content-Length: 0
Content-Type: text/html; charset=utf-8
Date: Fri, 28, Jun 2021 12:23:18 KST
Server: Werkzeug/0.14.1 Python/3.8.0
```

## PUT

POST 메소드와 비슷한 의미를 가지고 있는 메소드다. POST와 중복되는 의미이므로 데이터를 새로 생성하는 HTTP 요청을 보낼 때 굳이 PUT을 사용하지 않고 모든 데이터 생성 및 수정 관련한 요청은 다 POST로 통일해서 사용하는 시스템이 많아지고 있다.

## DELETE

데이터 삭제 요청을 보낼 떄 사용되는 메소드다.

# 자주 사용되는 HTTP Status Code와 Text

## 200 OK

HTTP 요청이 문제없이 성공적으로 잘 처리되었 때 보내는 status code다.

## 301 Method Permanently

요청을 보낸 엔드포인트의 URL 주소가 바뀌었다는 것을 나타내는 status code다.

## 400 Bad Request

HTTP 요청이 잘못된 요청일 떄 보내는 응답 코드다.

## 401 Unauthorized

HTTP 요청을 처리하기 위해서는 해당 요청을 보내는 주체의 신분 확인이 요구되나 확인할 수 없었을 떄 보내는 응답 코드다.

## 403 Forbidden

HTTP 요청을 보내는 주체가 해당 요청에 대한 권한이 없음을 나타내는 응답 코드다.

## 404 Not Found

HTTP 요청을 보내고자 하는 URI가 존재하지 않을 떄 보내는 응답 코드다.

## 500 Internal Server Error

내부 서버에 오류가 발생했다는 것을 알려주는 응답 코드다.

# API 엔드포인트 아키텍처 패턴

API의 엔드포인트 구조를 구현하는 방식에도 널리 알려지고 사용되는 패턴들이 있다. 크게 2가지가 있는데 하나는 REST 방식이고 다른 하나는 GraphQL 이다.

## RESTful HTTP API

RESTful HTTP API는 API 시스템을 구현하기 위한 아키텍처의 한 방식이다.
RESTful API는 API에서 전송하는 리소스를 URI로 표현하고 해당 리소스에 행하고자 하는 의도를 HTTP 메서드로 정의하는 방식이다.

장점은 자기 설명(self-descriptuveness) 이다. 즉 엔드포인트의 구조만 보더라도 해당 엔드포인트가 제공하는 리소스와 기능을 파악할 수 있다.

## GraphQL

한동안 REST 방식이 API를 구현하는 데 있어서 정석으로 여겨졌으나, 구조적으로 문제가 생이는 것들이 있다. 자주 생기는 문제는, API의 구조가 특정 클라이어트에 맞추어져서 다른 클라이언트에서 사용하기에 적합하지 않다.

이러한 문제를 해결하기 위해서 GraphQL를 만들어 된다. REST 방식의 API 보다 다르게 endpoint가 하나이다. 그러나 엔드포인트에 클라이언트가 필요한 것을 정의해서 요청하는 식이다.
