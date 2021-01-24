# Cross-Origin Resource Sharing

##  교차 출처 리소스 공유(CORS)란 무엇일까?

**교차 출처 리소스 공유(Cross-Origin Resource Sharing, Cors)** 는 추가 HTTP 헤더를 사용하여, 한 출처에서 실행중인 웹 애플리케이션이 다른 출처의 선책한 자원에 접근할 수 있는 권한을 부여하도록 브라우저에서 알려주는 체제이다. 웹 애플리케이션은 리소스가 자신의 출처(도메인, 프로토콜, 포트)와 다를 때 교차 출처 HTTP 요청을 실행한다.

```
Ex) `https://naver.com`의 프론트 엔드 JavaScript 코드가 XMLHttpRequest를 사용하여 `https://google.com/data.json`을 요청하는 경우
```

보안 상의 이유로, 브라우저는 스크립트에서 시작한 교차 출처 HTTP 요청을 제한한다. 예를들어 XMLHttpRequest와 FetchAPI는 동일 출처정책을 따른다. 이 API를 사용하는 애플리케이션은 자신과 동일한 출처에서만 리소스를 불러올 수 있다. 다른 출처의 리소스를 불러오려면 헤더에 추가적으로 CORS를 포함헤서 응답해야 한다.

##  탄생 배경

CORS가 나오게 된 이유는 바로 **동일 출처 정책 (Same Origin Policy)** 때문이다. SOP는 어떠한 출처에서 불러온 문서 또는 스크립트가  다른 출처에서 가져온 자원과 상호작용하는 것을 제한하는 정책이다. 서로 다른 Origin에서의 **자원 공유를 보안상의 이유로 제한**하는 것이다.

SOP는 **잠재적으로 해로울 수 있는 문서를 분리**함으로써 **공격받을 수 있을 경로**를 줄여준다.

## Cors principle

<img src="https://github.com/wookkl/study_backend/img/cors_principle.png" width="200px">

# References

_[https://developer.mozilla.org/ko/docs/Web/HTTP/CORS](https://developer.mozilla.org/ko/docs/Web/HTTP/CORS)_
