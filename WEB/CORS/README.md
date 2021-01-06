# 교차 출처 리소스 공유(CORS)란?

교차 출처 리소스 공유(Cross-Origin Resource Sharing, Cors)는 추가 HTTP 헤더를 사용하여, 한 출처에서 실행중인 웹 애플리케이션이 다른 출처의 선책한 자원에 접근할 수 있는 권한을 부여하도록 브라우저에서 알려주는 체제이다. 웹 애플리케이션은 리소스가 자신의 출처(도메인, 프로토콜, 포트)와 다를 때 교차 출처 HTTP 요청을 실행한다.

    Ex) `https://naver.com`의 프론트 엔드 JavaScript 코드가 XMLHttpRequest를 사용하여 `https://google.com/data.json`을 요청하는 경우

보안 상의 이유로, 브라우저는 스크립에서 시작한 교차 출처 HTTP 요청을 제한한다. 예를들어 XMLHttpRequest와 FetchAPI는 동일 출처정책을 따른다. 이 API를 사용하는 애플리케이션은 자신과 동일한 출처에서만 리소스를 불러올 수 있다. 다른 출처의 리소스를 불러오려면 헤더에 추가적으로 CORS를 포함헤서 응답해야 한다.

CORS 체제는 브라우저와 서버 간의 안전한 교차 출처 요청 및 데이터 전송을 지원한다. 최신 브라우저는 `XMLHttpRequest`와 `FetchAPI` 에서 COR를 사용하여 교차 출처 HTTP 요청의 위험을 예방한다.  
![CORS_Principle](../img/CORS_principle.png)

# CORS를 사용하는 요청들

다음과 같은 사이트간 HTTP 요청을 허용

1. `XMLHttpRequest` 와 `FetchAPI` 호출
2. @font-face에서 교차 도메인 폰트 사용시
3. WebGL 텍스쳐
4. 캔버스에 그린 이미지나 비디오 프레임

# 기능

CORS 표준은 웹 브라우저에서 어떤 정보를 읽는 것이 허용된 출처를 서버에 설명할 수 있는 HTTP헤더를 추가하면서 동작한다. 서버 데이터에 효과를 일으킬 수 있는 HTTP 요청 메서드(`GET`을 제외한 메서드)에 대히여, CORS 명세는 브라우저가 요청을 `OPTIONS` 메서드로 프리플라이트하여 지원하는 메서드를 요청한 후에, 서버가 허가한다면 그때 실제로 요청을 보내도록 요구한다. 또한 서버는 클라이언트에게 요청에 **인증정보** 를 함계 보내야 한다고 알려줄 수도 있다.

CORS 실패는 오류의 원인이지만, 보안때문에 자바스크립트에서 상세 정보를 볼 수 없고, 알 수 있는것은 단지 오류가 발생했다는 사실 뿐이다. 정확히 알아내려면 브라우저의 콘솔을 보아야 한다.

# 접근 제어 에제

## 단순요청(Simple requests)

일부 요청은 CORS 프리플라이트 하지 않는다. 이 단순한 요청은 다음 조건을 모두 충족하는 요청이다.

### 조건

- HTTP 요청 메서드
  - `GET`
  - `HEAD`
  - `POST`
- 사용자 에이전트가 자동으로 설정한 헤더 이외에, 수동으로 설정할 수 있는 헤더는 Fetch 명세에서 "CORS-safelisted request-header"로 정의한 헤더 뿐이다.
  - `Accept`
  - `Accept-Language`
  - `Content-Language`
  - `Content-Type`
  - `DPR`
  - `DownLik`
  - `Save-Data`
  - `Viewport-Width`
  - `Width`
- Content-Type은 다음의 값들만 허용한다.
  - `application/x-www-form-urlencoded`
  - `multipart/form-data`
  - `test/plain
- `XMlHttpRequestUpload` 객체는 이벤트 리스너가 등록되어 있지 않아야 한다. `XMlHttpRequest.upload` 프로퍼티를 사용하여 접근한다.
- 요청에 `ReadableStream` 객체가 사용되지 않아야한다.

### 예시

예를 들어, `https://naver.com`의 웹컨텐츠가 `https://google.com` 도메인의 컨텐츠를 호출하길 원하면 `naver.com`에 배포된 자바스크립트는

```javascript
const xhr = new XMLHttpRequest();

//실제 존재하는 url이 아닙니다
const url = "https://google.com/resources/public-data/";

xhr.open("GET", url);
xhr.onreadystatechange = someHandler;
xhr.send();
```

이다.
클라이언트와 서버간의 간단만 통신하고 CORS헤더를 사용하여 권한을 처리한다.

브라우저가 서버로 요청하는 내용을 보고, 서버의 응답을 확인한다.

**클라이언트 요청**

> GET /resources/public-data/ HTTP/1.1  
> Host: google.com  
> User-Agent: Mozilla/5.0 (Macintosh; Intel Mac OS X 10.14; rv:71.0) Gecko/20100101 Firefox/71.0  
> Accept: text/html,application/xhtml+xml,application/xml;q=0.9,_/_;q=0.8  
> Accept-Language: en-us,en;q=0.5  
> Accept-Encoding: gzip,deflate  
> Connection: keep-alive  
> **Origin: https://naver.com**

**서버 응답**

> HTTP/1.1 200 OK  
> Date: Mon, 01 Dec 2008 00:23:53 GMT  
> Server: Apache/2  
> **Access-Control-Allow-Origin: \***  
> Keep-Alive: timeout=2, max=100  
> Connection: Keep-Alive  
> Transfer-Encoding: chunked  
> Content-Type: application/xml  
> […XML Data…]

서버는 응답으로 `Access-Control-Allow-Origin` 헤더를 다시 전송한다. 가장 간단한 접근 제어 프로토콜은 `Origin` 헤더와 `Access-Control-Allow-Origin` 헤더를 사용하는 것이다. `Access-Control-Allow-Origin`는 **모든** 도메인에서 접근할 수 있다는 것을 의미한다.

> Access-Control-Allow-Origin: https://naver.com

이는 `https://naver.com`만 오직 cross-site 방식으로 리소스에 접근할 수 있다. 리소스에 대한 접근을 허용하려면 `Access-Control-Allow-Origin` 헤더에는 요청이 `Origin` 헤더에 전송된 값이 포함 되어야한다.

# References

_https://developer.mozilla.org/ko/docs/Web/HTTP/CORS_
