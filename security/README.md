# CSRF(Cross-site Request Forgery): 사이트간 요청 위조

웹 어플리케이션 취약점 중 하나로 사용자가 자신의 의지와는 무관하게 공격자가 의도한 행위를 특정 웹사이트에 요청하게 만드는 공격이다.

과정은

1. 사용자는 웹사이트에 로그인 하여 정상적은 쿠키를 발급받는다.
2. 공격자는 다음과 같은 링크를 이메일이나 게시판등을 통해 사용자에게 전달한다.
3. 공격용 HTML 이미지태그 예

```html
<img src="https://travel.service.com/travel_update?.src=Korea&.dst=Hell" />
```

4. 이용자가 공격용 페이지를 열면, 브라우저는 이미지 파일을 받아오기 위해 해당 src를 연다.
5. 이용자의 승인이나 의지 없이 src의 url을 열어버림으로써 출발지와 도착지가 korea->hell로 정해진다.

## CSRF 해결방법

- CSRF Token
